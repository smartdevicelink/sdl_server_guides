## Prerequisites
The following must be installed before installation of the Policy Server can begin:

| Project | Version |
|---------|---------|
| `Postgres` | 9.6+ |
| `Node.js` | 4.0.0+ |
| `NPM` | 3.0.0+ |

You must also acquire a set of SHAID API keys. These are made available to level 4 OEM members through the [developer portal](https://smartdevicelink.com/).

## Setup Guide
Download the project to your current directory.
```
git clone https://github.com/smartdevicelink/sdl_server.git
cd sdl_server
```
The recommended branch to use is master, which should be used by default. Install dependencies.
```
npm install
```

The new version of the Policy Server requires a SQL database. Currently the only supported implementation is PostgreSQL. This guide will not cover how to get one running.

Once you set up a database (locally or remotely) you'll need to supply the Policy Server with some environment variables. This Policy Server uses the [dotenv module](https://www.npmjs.com/package/dotenv), meaning you can write all your environment variables in a `.env` file located in the root directory of the Policy Server. The Policy Server will load the variables at `.env`. `.env` files will not be tracked by Git.

Here are the environment variables that will most likely be used:

* `POLICY_SERVER_HOST`: String. The hostname or public IP address which the server runs on.
* `POLICY_SERVER_PORT`: Integer. The port which the server runs on. It is optional and the default is 3000.
* `POLICY_SERVER_PORT_SSL`: Integer. The port which the server should listen for SSL connections on (typically 443). It is optional and the default is `null` (do not listen for SSL connections).
* `SSL_CERTIFICATE_FILENAME`: String. The filename of the SSL certificate located in `./customizable/ssl`. Required if a value is set for `POLICY_SERVER_PORT_SSL`.
* `SSL_PRIVATE_KEY_FILENAME`: String. The filename of the SSL certificate's private key located in `./customizable/ssl`. Required if a value is set for `POLICY_SERVER_PORT_SSL`.
* `SHAID_PUBLIC_KEY`: String. A public key given to you through the [developer portal](https://smartdevicelink.com/) that allows access to SHAID endpoints.
* `SHAID_SECRET_KEY`: String. A secret key given to you through the [developer portal](https://smartdevicelink.com/) that allows access to SHAID endpoints.
* `DB_USER`: String. The name of the user to allow the server to access the database
* `DB_DATABASE`: String. The name of the database where policy and app data is stored
* `DB_PASSWORD`: String. The password used to log into the database
* `DB_HOST`: String. The host name or IP address of the database
* `DB_PORT`: Integer. The port number of the database
* `CACHE_MODULE`: String. The name of the caching module to use. Currently supports null (no caching, default) or "redis".
* `CACHE_HOST`: String. The host name or IP address of the cache. Default null.
* `CACHE_PORT`: Integer. The port number of the cache. Default null.
* `CACHE_PASSWORD`: String. The password used to log into the cache. Default null.
* `SMTP_HOST`: String. The host name or IP address of an SMTP server to use for email notifications. A null value implies that outgoing emails are disabled. Default null.
* `SMTP_PORT`: Integer. The port number of the SMTP server. Default 25.
* `SMTP_USERNAME`: String. The username of the optional SMTP user. Default null.
* `SMTP_PASSWORD`: String. The password of the optional SMTP user. Default null.
* `SMTP_FROM`: String. The email address which emails are sent from. A null value implies that outgoing emails are disabled. Default null.
* `NOTIFY_APP_REVIEW_FREQUENCY`: String enum (DISABLED, REALTIME). The frequency of which outgoing emails should be sent to notify the OEM of new apps ready for review. Default DISABLED.
* `NOTIFY_APP_REVIEW_EMAILS`: String. A comma-separated list of email addresses to send an email to when new apps are ready for review. Default empty string.
* `AUTO_APPROVE_ALL_APPS`: String boolean ("true" or "false"). Whether or not to auto-approve all app versions received by SHAID (except for blacklisted apps). Default "false".
* `CA_PRIVATE_KEY_FILENAME`: String. Mandatory. The filename of your .key file generated, to be placed in `customizable/ca/`
* `CA_CERTIFICATE_FILENAME`: String. Mandatory. The filename of your .pem file generated, to be placed in `customizable/ca/`
* `CERTIFICATE_PASSPHRASE`: String. Mandatory. A secret password used for every certificate generated.
* `CERTIFICATE_COMMON_NAME`: String. Mandatory. Default information of the issuer's fully qualified domain name to secure.
* `PRIVATE_KEY_BITSIZE`: Integer. The size of the private keys generated. Default 2048.
* `PRIVATE_KEY_CIPHER`: String. The type of cipher to use for encryption/decryption. Defaults to "des3".
* `CERTIFICATE_COUNTRY`: String. Default information of the issuer's country (two-letter ISO code).
* `CERTIFICATE_STATE`: String. Default information of the issuer's state.
* `CERTIFICATE_LOCALITY`: String. Default information of the issuer's city.
* `CERTIFICATE_ORGANIZATION`: String. Default information of the issuer's legal company name.
* `CERTIFICATE_ORGANIZATION_UNIT`: String. Default information of the issuer's company's branch.
* `CERTIFICATE_EMAIL_ADDRESS`: String. Default information of the issuer's email address.
* `CERTIFICATE_HASH`: String. The cryptographic hash function to use. Default 'sha256'.
* `CERTIFICATE_DAYS`: Integer. The number of days until the certificate expires. Default 7.
* `ENCRYPTION_REQUIRED`: String boolean ("true" or "false"). Whether or not to require RPC encryption for auto-approved app versions. Default "false".

* `STAGING_PG_USER` **DEPRECATED**: String. The name of the user to allow the server access the database (staging mode)
* `STAGING_PG_DATABASE` **DEPRECATED**: String. The name of the database where policy and app data is stored (staging mode)
* `STAGING_PG_PASSWORD` **DEPRECATED**: String. The password used to log into the database (staging mode)
* `STAGING_PG_HOST` **DEPRECATED**: String. The host name or IP address of the database (staging mode)
* `STAGING_PG_PORT` **DEPRECATED**: Integer. The port number of the database (staging mode)
* `PRODUCTION_PG_USER` **DEPRECATED**: String. The name of the user to allow the server access the database (production mode)
* `PRODUCTION_PG_DATABASE` **DEPRECATED**: String. The name of the database where policy and app data is stored (production mode)
* `PRODUCTION_PG_PASSWORD` **DEPRECATED**: String. The password used to log into the database (production mode)
* `PRODUCTION_PG_HOST` **DEPRECATED**: String. The host name or IP address of the database (production mode)
* `PRODUCTION_PG_PORT` **DEPRECATED**: Integer. The port number of the database (production mode)

Production/Staging environment variables for the database are now deprecated. Please use the corresponding `DB_` values in place of them (ex. `DB_USER` instead of `PRODUCTION_PG_USER` or `STAGING_PG_USER`).

Once these environment variables are set, initialize the database. The database should be given the same name set in `DB_DATABASE`.

Using the createdb program that comes with the installation of PostgreSQL, for example:
`createdb policy_server`

The Policy Server comes with migration scripts that can be run using npm scripts. You can see a list of all the possible scripts by looking in `package.json`, but these are the most important ones:

* `start-server`: Runs the migration up script which initializes data in the database and starts the Policy Server
* `dev` or `start`: Starts the server with hot reloading so any changes made to the UI are instantly updated in the browser
* `build`: Generates a new staging/production build using webpack. This command should only be run if you made front-end modifications to the UI.
* `start-pg-staging` **DEPRECATED**: Runs the migration up script which initializes data in the database, sets the environment to `staging` and starts the Policy Server
* `start-pg-production` **DEPRECATED**: Runs the migration up script which initializes data in the database, sets the environment to `production` and starts the Policy Server
* `db-migrate-reset-pg-staging` **DEPRECATED**: Runs the migration down script which drops all the data and tables in the staging database

Production/Staging scripts are now deprecated. Please use `start-server` instead of `start-pg-staging` or `start-pg-production`.

Run the following command to finalize set up and start the server.

`npm run start-server`

Verify that it started properly by navigating to your configured host and port, or to the default address: <a href="http://localhost:3000/">`http://localhost:3000/`</a>

Now you have a Policy Server running!


* If you wish to enable caching with an unofficially supported datastore, you may create a custom cache module. Do so by creating a folder inside `custom/cache` with the name of your module. Put your implementation in a file named `index.js` inside of your module's folder. Your module should export the following functions:
    * `get(key, callback)`: Receives a value from the cache stored at key.
    * `set(key, value, callback)`: Sets a value in the cache stored at key.
    * `del(key, callback)`: Deletes a value from the cache stored at key.
    * `flushall(callback)`: Deletes all data previously set in the cache.
* Set your `CACHE_` environment variables to correspond with your new datastore solution and access information.
