# System image build with Oracle Database support


## Prerequisites

In order to work with Oracle Database for Red Hat 3scale API Management, you will need to create a custom image as Red Hat cannot distribute the binaries of Oracle Database client.

You **MUST** download the client (basic-lite or basic), the odbc driver and the sdk for **12.2** in [Oracle Technology Network](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html).

* Instant Client Package - Basic or Basic Lite
* Instant Client Package - SDK
* Instant Client Package - ODBC driver

Example:

    * instantclient-basiclite-linux.x64-12.2.0.1.0.zip or instantclient-basic-linux.x64-12.2.0.1.0.zip
    * instantclient-sdk-linux.x64-12.2.0.1.0.zip
    * instantclient-odbc-linux.x64-12.2.0.1.0-2.zip

### Oracle Database user SYSTEM password

We **need** the Oracle Database user `SYSTEM` password.
It will create the user and grant the appropriate roles and rights to the user connecting to the database.


## Instructions

1 - Place the downloaded files in the `oracle-client-files` directory.


2 - Download the productized amp.yml template.


3 - Run on your machine by replacing `WILDCARD_DOMAIN`


```
$ oc new-app -f build.yml
$ oc new-app -f amp.yml -p WILDCARD_DOMAIN=mydomain.com
```

4 - Run on your machine

:warning: In the next step please REPLACE `"<<<<SYSTEM_PASSWORD>>>>"` by the Oracle Database `SYSTEM` user password.

```
$ oc patch dc/system-app -p '[{"op": "add", "path": "/spec/strategy/rollingParams/pre/execNewPod/env/-", "value": {"name": "ORACLE_SYSTEM_PASSWORD", "value": "<<<<SYSTEM_PASSWORD>>>>"}}]' --type=json
$ oc patch dc/system-app -p '{"spec": {"strategy": {"rollingParams": {"post":{"execNewPod": {"env": [{"name": "ORACLE_SYSTEM_PASSWORD", "value": "<<<<SYSTEM_PASSWORD>>>>"}]}}}}}}'

```


5 - Change your DATABASE_URL to point to your Oracle Database (see last section). To do so:


:warning: In the next step please REPLACE `"<<<<DESIRED_VALUE_FOR_DATABASE_URL>>>>"` by the Oracle Database URL as specified in the last section

```
$ oc patch secret/system-database -p '{"stringData": {"URL": "<<<<DESIRED_VALUE_FOR_DATABASE_URL>>>>"}}'
```


6 - :warning: If you are using a Non Containerized database, apply one of the patches located in `workaround-non-cdb-2.4-upgrade/dockerfile.patch` or `workaround-non-cdb-2.5-upgrade/dockerfile.patch` depending on your 3scale version

For UNIX operating systems on 3scale 2.4:

```
patch -p1 < workaround-non-cdb-2.4-upgrade/dockerfile.patch
```

For UNIX operating systems on 3scale 2.5:

```
patch -p1 < workaround-non-cdb-2.5-upgrade/dockerfile.patch
```



7 - Then starts the build


```
$ oc start-build 3scale-amp-system-oracle --from-dir=.
```

## DATABASE_URL parameter specification

The `DATABASE_URL` parameter **MUST** follow this format `oracle-enhanced://${user}:${password}@${host}:${port}/${database}`

Example:

```shell
DATABASE_URL="oracle-enhanced://user:password@my-oracle-database.com:1521/threescalepdb"
```

## Oracle 12c Max VARCHAR2 Limits 

You need to switch ``VARCHAR2`` ``MAX_STRING_SIZE`` to ``EXTENDED`` before rollout of the modified system-amp pod otherwise you could get this error during the schema creation. More info https://docs.oracle.com/database/121/SQLRF/sql_elements001.htm#SQLRF55623


```shell
ActiveRecord::StatementInvalid: OCIError: ORA-00910: specified length too long for its datatype: CREATE TABLE "PROXIES" ("ID" NUMBER(38) NOT NULL PRIMARY KEY, "TENANT_ID" NUMBER(38), "SERVICE_ID" NUMBER(38), "ENDPOINT" VARCHAR2(255), "DEPLOYED_AT" DATE, "API_BACKEND" VARCHAR2(255), "AUTH_APP_KEY" VARCHAR2(255) DEFAULT 'app_key', "AUTH_APP_ID" VARCHAR2(255) DEFAULT 'app_id', "AUTH_USER_KEY" VARCHAR2(255) DEFAULT 'user_key', "CREDENTIALS_LOCATION" VARCHAR2(255) DEFAULT 'query' NOT NULL, "ERROR_AUTH_FAILED" VARCHAR2(255) DEFAULT 'Authentication failed', "ERROR_AUTH_MISSING" VARCHAR2(255) DEFAULT 'Authentication parameters missing', "CREATED_AT" DATE, "UPDATED_AT" DATE, "ERROR_STATUS_AUTH_FAILED" NUMBER(38) DEFAULT 403 NOT NULL, "ERROR_HEADERS_AUTH_FAILED" VARCHAR2(255) DEFAULT 'text/plain; charset=us-ascii' NOT NULL, "ERROR_STATUS_AUTH_MISSING" NUMBER(38) DEFAULT 403 NOT NULL, "ERROR_HEADERS_AUTH_MISSING" VARCHAR2(255) DEFAULT 'text/plain; charset=us-ascii' NOT NULL, "ERROR_NO_MATCH" VARCHAR2(255) DEFAULT 'No Mapping Rule matched' NOT NULL, "ERROR_STATUS_NO_MATCH" NUMBER(38) DEFAULT 404 NOT NULL, "ERROR_HEADERS_NO_MATCH" VARCHAR2(255) DEFAULT 'text/plain; charset=us-ascii' NOT NULL, "SECRET_TOKEN" VARCHAR2(255) NOT NULL, "HOSTNAME_REWRITE" VARCHAR2(255), "OAUTH_LOGIN_URL" VARCHAR2(255), "SANDBOX_ENDPOINT" VARCHAR2(255), "API_TEST_PATH" VARCHAR2(8192), "API_TEST_SUCCESS" NUMBER(1), "APICAST_CONFIGURATION_DRIVEN" NUMBER(1) DEFAULT 1 NOT NULL, "OIDC_ISSUER_ENDPOINT" VARCHAR2(255), "AUTHENTICATION_METHOD" VARCHAR2(255), "LOCK_VERSION" NUMBER(38) DEFAULT 0 NOT NULL, "POLICIES_CONFIG" CLOB)
stmt.c:243:in oci8lib_230.so
