# System image build with Oracle Database support


## Prerequisites

In order to work with Oracle Database for Red Hat 3scale API Management, you will need to create a custom image as Red Hat cannot distribute the binaries of Oracle Database client.

You **MUST** download the client (basic-lite or basic), the odbc driver and the sdk for **19.6** in [Oracle Technology Network](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html).

* Instant Client Package - Basic or Basic Lite
* Instant Client Package - SDK
* Instant Client Package - ODBC driver

Example:

    * instantclient-basiclite-linux.x64-19.6.0.0.0dbru.zip or instantclient-basic-linux.x64-19.6.0.0.0dbru.zip
    * instantclient-sdk-linux.x64-19.6.0.0.0dbru.zip
    * instantclient-odbc-linux.x64-19.6.0.0.0dbru.zip

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

On 3scale 2.6+, there is no need to apply any workaround patch for Non Containerized database

7 - Then starts the build


```
$ oc start-build 3scale-amp-system-oracle --from-dir=.
```

8 - Wait until the build completes. The available build names can be listed by
    executing `oc get builds` and then the state of a specific build can be
    verified with:
```
$ oc get build <build-name> -o jsonpath="{.status.phase}"
```

Wait until the build is in 'Complete' state

9 - Update the ImageChange triggers of the DeploymentConfigs that use the System image
    to use the new Oracle-based System image:

Save the current 3scale release in an environment variable:
```
$ export THREESCALE_RELEASE=2.11
```

Update system-app DeploymentConfig's ImageChangeTrigger:
```
$ oc set triggers dc/system-app --from-image=amp-system:${THREESCALE_RELEASE} --containers=system-master,system-developer,system-provider --remove
$ oc set triggers dc/system-app --from-image=amp-system:${THREESCALE_RELEASE}-oracle --containers=system-master,system-developer,system-provider
```


This triggers a redeployment of system-app DeploymentConfig. Wait until it is
redeployed, its corresponding new pods are ready, and the old ones terminated.


Update system-sidekiq DeploymentConfig's ImageChangeTrigger:
```
$ oc set triggers dc/system-sidekiq --from-image=amp-system:${THREESCALE_RELEASE} --containers=system-sidekiq,check-svc --remove
$ oc set triggers dc/system-sidekiq --from-image=amp-system:${THREESCALE_RELEASE}-oracle --containers=system-sidekiq,check-svc
```

This triggers a redeployment of system-sidekiq DeploymentConfig. Wait until it is
redeployed,  its corresponding new pods are ready, and the old ones terminated.

Update system-sphinx DeploymentConfig's ImageChangeTriger:
```
$ oc set triggers dc/system-sphinx --from-image=amp-system:${THREESCALE_RELEASE} --containers=system-sphinx,system-master-svc --remove
$ oc set triggers dc/system-sphinx --from-image=amp-system:${THREESCALE_RELEASE}-oracle --containers=system-sphinx,system-master-svc
```

This triggers a redeployment of system-sphinx DeploymentConfig. Wait until it is
redeployed, its corresponding new pods are ready, and the old ones terminated.


## DATABASE_URL parameter specification

The `DATABASE_URL` parameter **MUST** follow this format `oracle-enhanced://${user}:${password}@${host}:${port}/${database}`

Example:

```shell
DATABASE_URL="oracle-enhanced://user:password@my-oracle-database.com:1521/threescalepdb"
```

## Oracle 19c Max VARCHAR2 Limits

You need to configure your database properly in order to accomodate 3scale

```
ALTER SYSTEM SET max_string_size=extended SCOPE=SPFILE;
ALTER SYSTEM SET compatible='12.2.0.1' SCOPE=SPFILE;
```

