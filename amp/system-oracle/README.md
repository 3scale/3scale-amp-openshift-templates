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



6 - Then starts the build


```
$ oc start-build 3scale-amp-system-oracle --from-dir=.
```

## DATABASE_URL parameter specification

The `DATABASE_URL` parameter **MUST** follow this format `oracle-enhanced://${user}:${password}@${host}:${port}/${database}`

Example:

```shell
DATABASE_URL="oracle-enhanced://user:password@my-oracle-database.com:1521/threescalepdb"
```
