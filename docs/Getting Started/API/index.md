### Introduction
Below are the API routes that the Policy Server exposes. 

There may be CORS issues in the case where a separate web app needs to make API calls to the policy server, such as an HMI using the `applications/store` route. For cases like these, CORS is enabled. Preflight is also enabled for sufficiently complex POST requests. See below for the routes which have CORS or preflight enabled by default. The full list of routes and their middleware can also be seen in the project's `app/v1/app.js` file.

| Route               | CORS | Preflight |
|---------------------|------|-----------|
| POST /api/v1/staging/policy     | true | true      |
| POST /api/v1/production/policy  | true | true      |
| GET /api/v1/applications/store | true | false     |
| GET /api/v1/applications/store/staging | true | false     |

### `POST /api/v1/login`
If basic authentication is enabled, the Policy Server UI opens a login page on startup which will call this route. The Policy Server will then validate that the entered password matches the one set up by the server maintainer.

---
### `GET /api/v1/applications` & `GET /api/v1/groups` & `GET /api/v1/messages`
Retrieves information regarding applications, functional groups, or consumer friendly messages. An **id** (or additionally **uuid** for applications) can be specified so as to retrieve information for a specific item. Functional groups and consumer messages can be set to return templates containing all necessary information on that item being stored in the database. Applications can be filtered by approval status. If no parameters are specified `/applications` will return the latest version of each app, `/groups` and `/messages` will return the latest version of all functional groups or consumer messages in either production or staging mode.

---
### `POST /api/v1/applications/action`
Updates an application's approval status. In the future this route will also notify the app's developer via email of the change in approval status.

---
### `POST /api/v1/applications/auto`
If an application has been set to automatically approve all future updates then this route will validate the app uuid and update the approval status. In the future this route will also notify the app's developer via email of the change in approval status.

---
### `POST /api/v1/applications/administrator`
This route updates whether an app will have access to administrator functional groups.

---
### `POST /api/v1/applications/passthrough`
This route updates whether an app will be able to send unknown RPCs through App Service RPC Passsthrough.

---
### `POST /api/v1/applications/hybrid`
This route updates the hybrid preference of an app.

---
### `PUT /api/v1/applications/rpcEncryption`
This route updates whether an app should have RPC Encryption enabled.

---
### `PUT /api/v1/applications/service/permission`
This route modifies the App Services permissions of an application.

---
### `POST & GET /api/v1/applications/certificate/get`
This route queries the Policy Server database for an app's certificate and returns it, unless it's expired. If it is expired a 400 response is returned. Either appId or AppId is required in the query or the json body of the request.

Example requests:
```
GET /api/v1/applications/certificate/get?appId=31cc4209-79e7-4704-9ec4-3b485d3eeb93
```
OR
```
POST /api/v1/applications/certificate/get
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

---
### `POST /api/v1/applications/certificate`
This route updates the pkcs12 certificate of an application in the database.

---
### `GET /api/v1/applications/groups`
Returns the functional groups for which a given application has access.

---
### `PUT /api/v1/applications/groups`
Updates the functional groups for which a given application has access.

---
### `GET /api/v1/applications/store`
Retrieves approved, embedded application information, filterable by `uuid` or by `transport_type`. The possible values for `transport_type` are `webengine` and `websocket`. The return object includes app bundle information such as the location of the bundle and its file size, compressed and uncompressed. The logic of where to store these app packages is customizable by the policy server. See the `customizable/webengine-bundle/index.js` file for details.

---
### `GET /api/v1/applications/store/staging`
Does the same thing as /api/v1/applications/store, but returns apps whose approval statuses are in staging.

---
### `POST /api/v1/webhook`
This is the route that should be specified on a company's page on the SDL Developer Portal (in the box titled Webhook URL under Company Info) to be hit by the SHAID server when an app has been updated.

---
### `POST /api/v1/staging/policy` & `POST /api/v1/production/policy`
These are the routes sdl_core's default Policy Table should use when requesting a Policy Table update with either `/staging` or `/production` specified.
Given a "shortened" Policy Table, the Policy Server will use that information to automatically construct a full Policy Table response and return it to the requester.

---
### `GET /api/v1/policy/preview`
This is the route hit by the Policy Server UI requesting a preview of the Policy Table. A variable **environment** indicates whether it is to be staging or production.

---
### `POST /api/v1/policy/apps`
The Policy Server UI makes a request to this route which returns an example Policy Table segment for a particular app.

---
### `POST /api/v1/permissions/update`
The route updates the available permissions and permission relationships from SHAID.

---
### `GET /api/v1/permissions/unmapped`
This route returns a list of permissions that are currently not attributed to any functional groups.

---
### `POST /api/v1/groups` & `POST /api/v1/messages`
These routes are hit by the Policy Server UI to update a functional group's/consumer message's information or to change its deleted status.

---
### `GET /api/v1/groups/names` & `GET /api/v1/messages/names`
These routes return the names of all functional groups or consumer friendly messages recognized by the Policy Server.

---
### `POST /api/v1/groups/promote` & `POST /api/v1/messages/promote`
These routes are hit by the Policy Server UI to promote a functional group or consumer message from staging to production. If the functional group has a user consent prompt associated with it then the consent prompt must be promoted to production before promoting the functional group.

---
### `POST /api/v1/messages/update`
This route updates the Policy Server's list of languages.

---
### `GET /api/v1/module`
This route will return either the staging or production module config object.

---
### `POST /api/v1/module`
This route will update the staging module config object on record.

---
### `POST /api/v1/module/promote`
This route will promote the current staging module config to production.

---
### `POST /api/v1/security/certificate`
This route will return a PEM certificate. It is used by the UI when generating app certificates and must be provided a private key in order to function.

---
### `POST /api/v1/security/private`
This route will return a RSA private key. It is used by the UI when generating an application's private keys.

---
### `POST /api/v1/vehicle-data`
This route will add or update a custom vehicle data item.

---
### `GET /api/v1/vehicle-data`
This route will return a list of custom vehicle data items filtered by status and optionally by id.

---
### `POST /api/v1/vehicle-data/promote`
This route will promote the custom vehicle data on staging to production.

---
### `GET /api/v1/vehicle-data/type`
This route will return a list of all the data types and custom vehicle data parameter types on record.

---
## User Interface Pages
These are API routes that are accessed by the Policy Server user interface.

---
### `/applications`
The [Applications](/guides/sdl-server/user-interface/applications) page.

---
### `/applications/:id`
The App Details page with information regarding an app specified by the **id**. The Applications page documentation contains more information pertaining to this page.

---
### `/policytable`
The [View Policy Table](/guides/sdl-server/user-interface/view-policy-table) page.

---
### `/functionalgroups`
The [Functional Groups](/guides/sdl-server/user-interface/messages-and-functional-groups) page.

---
### `/functionalgroups/manage`
The Functional Group Details page with information regarding a functional group that is specified by an **id**. The Functional Groups page documentation contains more information pertaining to this page.

---
### `/consumermessages`
The [Consumer Friendly Messages](/guides/sdl-server/user-interface/messages-and-functional-groups) page.

---
### `/consumermessages/manage`
The Consumer Message Details page with information regarding a consumer message that is specified by an **id**. The Consumer Messages page documentation contains more information pertaining to this page.

---
### `/vehicledata`
The [Custom Vehicle Data](/guides/sdl-server/user-interface/custom-vehicle-data) page.

---
### `/moduleconfig`
The [Module Config](/guides/sdl-server/user-interface/module-config) page.

---

### `/about`
The [About](/guides/sdl-server/user-interface/about) page.