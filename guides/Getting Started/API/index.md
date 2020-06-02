These are the API routes that the Policy Server exposes.

### `GET /applications` & `GET /groups` & `GET /messages`
Retrieves information regarding applications, functional groups, or consumer friendly messages. An **id** (or additionally **uuid** for applications) can be specified so as to retrieve information for a specific item. Functional groups and consumer messages can be set to return templates containing all necessary information on that item being stored in the database. Applications can be filtered by approval status. If no parameters are specified `/applications` will return the latest version of each app, `/groups` and `/messages` will return the latest version of all functional groups or consumer messages in either production or staging mode.

### `POST /applications/action`
Updates an application's approval status. In the future this route will also notify the app's developer via email of the change in approval status.

### `POST /applications/auto`
If an application has been set to automatically approve all future updates then this route will validate the app uuid and update the approval status. In the future this route will also notify the app's developer via email of the change in approval status.

### `POST /staging/policy` & `POST /production/policy`
These are the routes sdl_core's default Policy Table should use when requesting a Policy Table update with either `/staging` or `/production` specified.
Given a "shortened" Policy Table, the Policy Server will use that information to automatically construct a full Policy Table response and return it to the requester.

### `GET /policy/preview`
This is the route hit by the Policy Server UI requesting a preview of the Policy Table. A variable **environment** indicates whether it is to be staging or production.

### `POST /policy/apps`
The Policy Server UI makes a request to this route which returns an example Policy Table segment for a particular app.

### `POST /webhook`
This is the route that should be specified on a company's page on the SDL Developer Portal (in the box titled Webhook URL under Company Info) to be hit by the SHAID server when an app has been updated.

### `POST /permissions/update`
This route queries SHAID for an update to the list of app permissions in the Policy Server.

### `GET /permissions/unmapped`
This route returns a list of permissions that are currently not attributed to any functional groups.

### `POST /groups` & `POST /messages`
These routes are hit by the Policy Server UI to update a functional group's/consumer message's information or to change its deleted status.

### `POST /groups/promote` & `POST /messages/promote`
These routes are hit by the Policy Server UI to promote a functional group or consumer message from staging to production. If the functional group has a user consent prompt associated with it then the consent prompt must be promoted to production before promoting the functional group.

### `POST /messages/update`
This route updates the Policy Server's list of languages.

### `GET /applications/store`
Retrieves approved, embedded application information, filterable by `uuid` or by `transport_type`. The possible values for `transport_type` are `webengine` and `websocket`. The return object includes app bundle information such as the location of the bundle and its file size, compressed and uncompressed. The logic of where to store these app packages is customizable by the policy server. See the `customizable/webengine-bundle/index.js` file for details.

### `POST & GET /applications/certificate/get`
This route queries the Policy Server database for an app's certificate and returns it, unless it's expired. If it is expired a 400 response is returned. Either appId or AppId is required in the query or the json body of the request.

Example requests:
```
/applications/certificate/get?appId=31cc4209-79e7-4704-9ec4-3b485d3eeb93 GET

/applications/certificate/get POST
{
    appId: 31cc4209-79e7-4704-9ec4-3b485d3eeb93
}
```

Response:
```json
{
    "meta": {
        "request_id": "427a7fb4-f2f1-44d6-8c2b-e7d927790960",
        "code": 200,
        "message": null
    },
    "data": {
        "certificate": "MIIKMQIBAzCCCf..."
    }
}
```

The certificate is a Base64 encoded string containing the pkcs12 certificate. This contains the certificate and private key and can be read using an openssl library with the password provided as ```CERTIFICATE_PASSPHRASE``` in your server's .env settings.

Example using openssl (note that the cert is a Base64 string and the ```CERTIFICATE_PASSPHRASE`` is used to read the pkcs12 certificate):
```
echo "MIIKMQIBAzCCCf..." | base64 -D > app-cert.p12 && openssl pkcs12 -nokeys -in app-cert.p12 -passin pass:CERTIFICATE_PASSPHRASE
```


## UI endpoints
These are API routes that are accessed by the Policy Server user interface.

### `/applications`
The [Applications](/docs/sdl-server/master/user-interface/applications) page.
### `/applications/:id`
The App Details page with information regarding an app specified by the **id**. The Applications page documentation contains more information pertaining to this page.
### `/policytable`
The [View Policy Table](/docs/sdl-server/master/user-interface/view-policy-table) page.
### `/functionalgroups`
The [Functional Groups](/docs/sdl-server/master/user-interface/messages-and-functional-groups) page.
### `/functionalgroups/manage`
The Functional Group Details page with information regarding a functional group that is specified by an **id**. The Functional Groups page documentation contains more information pertaining to this page.
### `/consumermessages`
The [Consumer Friendly Messages](/docs/sdl-server/master/user-interface/messages-and-functional-groups) page.
### `/consumermessages/manage`
The Consumer Message Details page with information regarding a consumer message that is specified by an **id**. The Consumer Messages page documentation contains more information pertaining to this page.
