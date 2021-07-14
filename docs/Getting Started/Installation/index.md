## Prerequisites
The following must be installed before installation of the Policy Server can begin:

| Project | Version |
|---------|---------|
| `Postgres` | 9.6+ |
| `Node.js` | 4.0.0 - 12.22.0 |
| `NPM` | 3.0.0+ |

Note the maximum Node version. **The policy server will not work on Node versions 13 or higher.**

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

The Policy Server requires a SQL database, and currently the only supported implementation is PostgreSQL. In the next section, we will cover how to get one running locally.

## PostgreSQL Database
To install PostgreSQL on a Mac with Homebrew, run the following command in a Terminal window:
```
brew install postgresql
```

Then run the following command to start PostgreSQL, and ensure that you won't need to start it again in case your system resets:
```
pg_ctl -D /usr/local/var/postgres start && brew services start postgresql
```

You can run the following command to know if you have PostgreSQL and also check that you are running the most recent version:
```
psql -V
```

In order to start creating users and databases, you will have to log in to PostgreSQL. It comes with a `postgres` user that has no password by default. Run the following command to log in as the `postgres` user:
```
psql -U postgres
```

You should now be in the postgres command-line interface. You can type `help` to get more info. If you want to continue using the `postgres` user, you can add a password with the following command:
```
ALTER USER postgres WITH PASSWORD '<password>';
```

If you want to create a new user, run the following commands to create one with a password and give them super user access:
```
CREATE USER <username> WITH PASSWORD '<password>';
ALTER USER <username> WITH SUPERUSER;
```

Alternatively you can use the [GRANT](https://tableplus.com/blog/2018/04/postgresql-how-to-grant-access-to-users.html) command to limit the user's permissions. In the future, you can log in to PostgreSQL using this new user. Next, you'll need to run the following command to add a new database for the Policy Server to manage:
```
CREATE DATABASE <database_name>;
```

This database will be where the Policy Server stores all of its data pertaining to policy table generation. Remember to save your PostgreSQL username, password, and database name so you can use them in the next section. To exit the PostgreSQL CLI, simply type `quit` and hit Enter.

## Environment Variables

Once you set up a database (locally or remotely) you'll need to supply the Policy Server with some environment variables. This Policy Server uses the [dotenv module](https://www.npmjs.com/package/dotenv), meaning you can write all your environment variables in a `.env` file located in the root directory of the Policy Server. The Policy Server will load the variables at `.env`. `.env` files will not be tracked by Git.

There are several settings that can be configured for Policy Server usage. See below for explanations on the purpose of each of them.

### Basic Environment Variables

| Name               | Type   | Example          | Description                                                               |
|--------------------|--------|------------------|---------------------------------------------------------------------------|
| POLICY_SERVER_HOST | String | testing.com      | The hostname or public IP address which the server runs on                |
| POLICY_SERVER_PORT | Number | 3000             | The port which the server runs on. It is optional and the default is 3000 |
| DB_USER            | String | postgres         | The name of the user to allow the server to access the database           |
| DB_DATABASE        | String | postgres         | The name of the database where policy and app data is stored              |
| DB_PASSWORD        | String | password         | The password used to log into the database                                |
| DB_HOST            | String | rds-database.com | The host name or IP address of the database                               |
| DB_PORT            | Number | 5432             | The port number of the database                                           |

### SHAID Environment Variables
| Name             | Type   | Description                                                                                                                             |
|------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------|
| SHAID_PUBLIC_KEY | String | A public key given to you through the [developer portal](https://smartdevicelink.com/) that allows access to SHAID endpoints.           |
| SHAID_SECRET_KEY | String | A secret key given to you through the [developer portal](https://smartdevicelink.com/) that allows access to SHAID endpoints.           |
| SHAID_URL        | String | The location of the SHAID server. The default value will query the production SHAID server. It is not recommended to change this value. |

### Caching Environment Variables
| Name           | Type   | Example        | Description                                                                                     |
|----------------|--------|----------------|-------------------------------------------------------------------------------------------------|
| CACHE_MODULE   | String | Redis          | The name of the caching module to use. Currently supports null (no caching, default) or "redis" |
| CACHE_HOST     | String | redis-host.com | The host name or IP address of the cache server                                                 |
| CACHE_PORT     | Number | 6379           | The port number of the cache server                                                             |
| CACHE_PASSWORD | String | password       | The password used to log into the cache server                                                  |

### Emailing Environment Variables
| Name                        | Type                               | Example                               | Description                                                                                                                          |
|-----------------------------|------------------------------------|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| SMTP_HOST                   | String                             | smpt-host.com                         | The host name or IP address of an SMTP server to use for email notifications. A null value implies that outgoing emails are disabled |
| SMTP_PORT                   | Number                             | 25                                    | The port number of the SMTP server. The default is 25                                                                                |
| SMTP_USERNAME               | String                             | smtp                                  | The username of the optional SMTP user                                                                                               |
| SMTP_PASSWORD               | String                             | password                              | The password of the optional SMTP user                                                                                               |
| SMTP_FROM                   | String                             | example@email.com                     | The email address which emails are sent from. A null value implies that outgoing emails are disabled                                 |
| NOTIFY_APP_REVIEW_FREQUENCY | String Enum (DISABLED, REALTIME)   | REALTIME                              | The frequency of which outgoing emails should be sent to notify the OEM of new apps ready for review. The default is DISABLED        |
| NOTIFY_APP_REVIEW_EMAILS    | String with comma-separated values | example1@email.com,example2@email.com | A comma-separated list of email addresses to send an email to when new apps are ready for review                                     |

### Mandatory Certificate and Encryption Environment Variables
| Name                    | Type   | Example       | Description                                                                  |
|-------------------------|--------|---------------|------------------------------------------------------------------------------|
| CA_PRIVATE_KEY_FILENAME | String | CA.key        | The filename of your .key file generated, to be placed in `customizable/ca/` |
| CA_CERTIFICATE_FILENAME | String | CA.pem        | The filename of your .pem file generated, to be placed in `customizable/ca/` |
| CERTIFICATE_PASSPHRASE  | String | password      | A secret password used for every certificate generated                       |
| CERTIFICATE_COMMON_NAME | String | *.company.com | Default information of the issuer's fully qualified domain name to secure    |

### Optional Certificate and Encryption Environment Variables
| Name                          | Type    | Example                         | Description                                                                                                                                                  |
|-------------------------------|---------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| POLICY_SERVER_PORT_SSL        | Number  | 443                             | The port which the server should listen for SSL connections on (typically 443). It is optional and the default is `null` (do not listen for SSL connections) |
| SSL_CERTIFICATE_FILENAME      | String  | file.pem                        | The filename of the SSL certificate located in `./customizable/ssl`. Required if a value is set for `POLICY_SERVER_PORT_SSL`                                 |
| SSL_PRIVATE_KEY_FILENAME      | String  | file.key                        | The filename of the SSL certificate's private key located in `./customizable/ssl`. Required if a value is set for `POLICY_SERVER_PORT_SSL`                   |
| PRIVATE_KEY_BITSIZE           | Number  | 2048                            | The size of the private keys generated. Default 2048                                                                                                         |
| PRIVATE_KEY_CIPHER            | String  | des3                            | The type of cipher to use for encryption/decryption. Defaults to "des3"                                                                                      |
| CERTIFICATE_COUNTRY           | String  | US                              | Default information of the issuer's country (two-letter ISO code)                                                                                            |
| CERTIFICATE_STATE             | String  | Michigan                        | Default information of the issuer's state                                                                                                                    |
| CERTIFICATE_LOCALITY          | String  | Royal Oak                       | Default information of the issuer's city                                                                                                                     |
| CERTIFICATE_ORGANIZATION      | String  | Livio                           | Default information of the issuer's legal company name                                                                                                       |
| CERTIFICATE_ORGANIZATION_UNIT | String  | Human Resources                 | Default information of the issuer's company's branch                                                                                                         |
| CERTIFICATE_EMAIL_ADDRESS     | String  | example@email.com               | Default information of the issuer's email address                                                                                                            |
| CERTIFICATE_HASH              | String  | sha256                          | The cryptographic hash function to use. Defaults to 'sha256'                                                                                                 |
| CERTIFICATE_DAYS              | Number  | 7                               | The number of days until the certificate expires. Defaults to 7                                                                                              |
| ENCRYPTION_REQUIRED           | Boolean | true | Whether or not to require RPC encryption for auto-approved app versions. Defaults to "false"                                                                 |
| MODULE_CONFIG_ENCRYPT_CERT_BUNDLE           | Boolean | true | Whether to package the module config's certificate and private key into a pkcs12 bundle string using the CERTIFICATE_PASSPHRASE. If false (default), it will just be a concatenation of the certificate and the private key                                                                 |

### Miscellaneous Environment Variables
| Name                  | Type    | Example | Description                                                                                                          |
|-----------------------|---------|---------|----------------------------------------------------------------------------------------------------------------------|
| AUTO_APPROVE_ALL_APPS | Boolean | true    | Whether or not to auto-approve all app versions received by SHAID (except for blacklisted apps). Defaults to "false" |

### Deprecated Environment Variables
| Name                   | Type   | Description                                                                    |
|------------------------|--------|--------------------------------------------------------------------------------|
| STAGING_PG_USER        | String | The name of the user to allow the server access the database (staging mode)    |
| STAGING_PG_DATABASE    | String | The name of the database where policy and app data is stored (staging mode)    |
| STAGING_PG_PASSWORD    | String | The password used to log into the database (staging mode)                      |
| STAGING_PG_HOST        | String | The host name or IP address of the database (staging mode)                     |
| STAGING_PG_PORT        | Number | The port number of the database (staging mode)                                 |
| PRODUCTION_PG_USER     | String | The name of the user to allow the server access the database (production mode) |
| PRODUCTION_PG_DATABASE | String | The name of the database where policy and app data is stored (production mode) |
| PRODUCTION_PG_PASSWORD | String | The password used to log into the database (production mode)                   |
| PRODUCTION_PG_HOST     | String | The host name or IP address of the database (production mode)                  |
| PRODUCTION_PG_PORT     | Number | The port number of the database (production mode)                              |

Production/Staging environment variables for the database are now deprecated. Please use the corresponding `DB_` values in place of them (ex. `DB_USER` instead of `PRODUCTION_PG_USER` or `STAGING_PG_USER`).

The Policy Server comes with migration scripts that can be run using npm scripts. You can see a list of all the possible scripts by looking in `package.json`, but these are the most important ones:

* `start-server`: Runs the migration up script which initializes data in the database and starts the Policy Server
* `dev` or `start`: Starts the server with hot reloading so any changes made to the UI are instantly updated in the browser
* `build`: Generates a new staging/production build using webpack. This command should only be run if you made front-end modifications to the UI.
* `lint`: Parses the Policy Server code and checks for syntactical or stylistic errors.
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
