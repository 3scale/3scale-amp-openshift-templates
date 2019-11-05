FROM amp-system:2.6

USER root

COPY ./oracle-client-files/instantclient-basic*-linux.x64-12.2.*.zip \
     ./oracle-client-files/instantclient-sdk-linux.x64-12.2.*.zip \
     ./oracle-client-files/instantclient-odbc-linux.x64-12.2.*.zip \
     /opt/oracle/

COPY ./oracle-client-files/vendor/cache/activerecord-oracle_enhanced-adapter-1.6.9.gem /opt/system/vendor/cache/
COPY Gemfile.oracle Gemfile.oracle.lock /opt/system/

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2/ \
    ORACLE_HOME=/opt/oracle/instantclient_12_2/ \
    DB=oracle \
    TZ=utc \
    NLS_LANG=AMERICAN_AMERICA.UTF8


RUN unzip /opt/oracle/instantclient-basic*-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && unzip /opt/oracle/instantclient-sdk-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && unzip /opt/oracle/instantclient-odbc-linux.x64-12.2.*.zip -d /opt/oracle/ \
 && rm -f /opt/oracle/*.zip \
 && (cd /opt/oracle/instantclient_12_2/ && ln -s libclntsh.so.12.1 libclntsh.so)

RUN source /opt/app-root/etc/scl_enable \
 && BUNDLE_GEMFILE=Gemfile.oracle DB=oracle bundle install --verbose --deployment --local


USER 1001
