# WebEngine Support

With the introduction of WebEngine applications, the Policy Server requires additional setup to be able to support them. This is because WebEngine apps need to be uploaded and servable for execution by supported HMIs, and the Policy Server provides the information necessary for downloading these app bundles. The implementation details are up to the Policy Server maintainer, as the method of hosting these WebEngine app bundles may depend on the environment and manner in which the Policy Server is running. 

### Getting Started
The entrypoint for the custom implementation starts in the `customizable/webengine-bundle/index.js` file in the project. In the single function stub `handleBundle` the URL of the app bundle is passed in. The goal is for the Policy Server to download the bundle from the passed in URL, extract the bundle to get the compressed and uncompressed file size data, and to host it in a publicly accessible location. The new URL for the WebEngine app bundle and its size information is expected to be returned in the `cb` argument for the `handleBundle` function, and that information will automatically be reflected in future calls to the `/api/v1/applications/store` route. Check the `customizable/webengine-bundle/index.js` file comments for specifics.

It is recommended that the app bundles are hosted in a dedicated online file-sharing service such as AWS's S3 buckets. These URLs are expected to be persistent and unchanging, even after Policy Server restarts or migrations. 

### S3 Storage Code Example
You may use this code snippet for reference on how to implement the `handleBundle` function. This implementation stores the app bundles on an S3 bucket, and assumes that your computer's credentials are set up to be authenticated with AWS, and that you have installed the `node-stream-zip` and `aws-sdk` node modules to the Policy Server.

```js
// skeleton function for customized downloading and extracting of package information
const request = require('request');
const fs = require('fs');
const UUID = require('uuid');
const AWS = require('aws-sdk');
const StreamZip = require('node-stream-zip');
AWS.config.update({region: 'us-east-1'});
const BUCKET_NAME = 'webengine-bundles';

/**
 * asynchronous function for downloading the bundle from the given url and extracting its size information
 * @param package_url - a publicly accessible external url that's used to download the bundle onto the Policy Server
 * @param cb - a callback function that expects two arguments
 *      if there was a failure in the process, it should be sent as the first argument. the Policy Server will log it
 *      the second argument to return must follow the formatted object below
 *      {
 *          url: the Policy Server should save a copy of the app bundle somewhere publicly accessible
 *              this url must be a full resolved url
 *          size_compressed_bytes: the number of bytes of the compressed downloaded bundle
 *          size_decompressed_bytes: the number of bytes of the extracted downloaded bundle
 *      }
 */
exports.handleBundle = function (package_url, cb) {
    let compressedSize = 0;
    let bucketUrl = '';
    const TMP_FILE_NAME = `${UUID.v4()}.zip`;

    // create a new bucket if it doesn't already exist
    new AWS.S3().createBucket({Bucket: BUCKET_NAME, ACL: 'public-read'}, err => {
        // OperationAborted errors are expected, as we are potentially 
        // calling this API multiple times simultaneously
        if (err && err.code !== 'OperationAborted') {
            return cb(err);
        }
        // read the URL and save it to a buffer variable
        readUrlToBuffer(package_url)
            .then(zipBuffer => { // submit the file contents to S3
                compressedSize = zipBuffer.length;
                const randomString = UUID.v4();
                const fileName = `${randomString}.zip`;
                bucketUrl = `https://${BUCKET_NAME}.s3.amazonaws.com/${fileName}`;
                // make the bundle publicly accessible
                const objectParams = {Bucket: BUCKET_NAME, ACL: 'public-read', Key: fileName, Body: zipBuffer};
                // Create object upload promise
                return new AWS.S3().putObject(objectParams).promise();
            })
            .then(() => { // unzip the contents of the bundle to get its uncompressed data information
                return streamUrlToTmpFile(bucketUrl, TMP_FILE_NAME);
            })
            .then(() => {
                return unzipAndGetUncompressedSize(TMP_FILE_NAME);
            })
            .then(uncompressedSize => {
                // delete the tmp zip file
                fs.unlink(TMP_FILE_NAME, () => {
                    // all the information has been collected
                    cb(null, {
                        url: bucketUrl,
                        size_compressed_bytes: compressedSize,
                        size_decompressed_bytes: uncompressedSize
                    });
                });
            })
            .catch(err => {
                // delete the tmp zip file
                fs.unlink(TMP_FILE_NAME, () => {
                    cb(err);
                });
            });
    });
}

function unzipAndGetUncompressedSize (fileName) {
    let uncompressedSize = 0;

    return new Promise((resolve, reject) => {
        const zip = new StreamZip({
            file: fileName,
            skipEntryNameValidation: true
        });
        zip.on('ready', () => {
            // iterate through every unzipped entry and count up the file sizes
            for (const entry of Object.values(zip.entries())) {
                if (!entry.isDirectory) {
                    uncompressedSize += entry.size;
                }
            }
            // close the file once you're done
            zip.close()
            resolve(uncompressedSize);
        });

        // Handle errors
        zip.on('error', err => { reject(err) });
    });
}

function streamUrlToTmpFile (url, fileName) {
    return new Promise((resolve, reject) => {
        request(url)
            .pipe(fs.createWriteStream(fileName))
            .on('close', resolve);
    });
}

function readUrlToBuffer (url) {
    return new Promise((resolve, reject) => {
        let zipBuffer = [];

        request(url)
            .on('data', data => {
                zipBuffer.push(data);
            })
            .on('close', function () { // file fully downloaded
                // put the zip contents to a buffer 
                resolve(Buffer.concat(zipBuffer));
            });
    })
}

```