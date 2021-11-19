FROM registry.redhat.io/3scale-amp2/system-rhel7:3scale2.11

USER root

COPY ./oracle-client-files/instantclient-basic*-linux.*.zip \
     ./oracle-client-files/instantclient-sdk-linux.*.zip \
     ./oracle-client-files/instantclient-odbc-linux.*.zip \
     /opt/system/vendor/oracle/

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient/:$LD_LIBRARY_PATH \
    ORACLE_HOME=/opt/oracle/instantclient/ \
    DB=oracle \
    TZ=utc \
    NLS_LANG=AMERICAN_AMERICA.UTF8

RUN ./script/oracle/install-instantclient-packages.sh \
 && source /opt/app-root/etc/scl_enable \
 && DB=oracle bundle install --local --deployment --jobs $(grep -c processor /proc/cpuinfo) --retry=5


USER 1001
