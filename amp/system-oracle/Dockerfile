FROM amp-system:2.5.0

USER root

COPY ./oracle-client-files/instantclient-basic*-linux.x64-12.2.*.zip \
     ./oracle-client-files/instantclient-sdk-linux.x64-12.2.*.zip \
     ./oracle-client-files/instantclient-odbc-linux.x64-12.2.*.zip \
     /opt/oracle/

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2/ \
    ORACLE_HOME=/opt/oracle/instantclient_12_2/ \
    DB=oracle \
    TZ=utc \
    NLS_LANG=AMERICAN_AMERICA.UTF8


RUN unzip /opt/oracle/instantclient-basic*-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && unzip /opt/oracle/instantclient-sdk-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && unzip /opt/oracle/instantclient-odbc-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && rm -f /opt/oracle/*.zip \
 && (cd /opt/oracle/instantclient_12_2/ && ln -s libclntsh.so.12.1 libclntsh.so) \
 && source /opt/app-root/etc/scl_enable \
 && DB=oracle bundle install --local --deployment --jobs $(grep -c processor /proc/cpuinfo) --retry=5


USER 1001
