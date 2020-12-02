FROM registry-proxy.engineering.redhat.com/rh-osbs/3scale-amp2-system-rhel7:1.14.0-7

USER root

COPY ./oracle-client-files/instantclient-basic*-linux.x64*.zip \
     ./oracle-client-files/instantclient-sdk-linux.x64*.zip \
     ./oracle-client-files/instantclient-odbc-linux.x64*.zip \
     /opt/system/vendor/oracle/

ENV LD_LIBRARY_PATH=/opt/rh/rh-nodejs10/root/usr/lib64:/opt/rh/rh-ruby25/root/usr/lib64:/opt/rh/rh-nodejs10/root/usr/lib64:/opt/oracle/instantclient/ \
    ORACLE_HOME=/opt/oracle/instantclient/ \
    DB=oracle \
    TZ=utc \
    NLS_LANG=AMERICAN_AMERICA.UTF8 \
    BASH_ENV=/opt/app-root/etc/scl_enable \
    ENV=/opt/app-root/etc/scl_enable \
    PROMPT_COMMAND=". /opt/app-root/etc/scl_enable" \
    RAILS_ENV=production \
    SAFETY_ASSURED=1 \
    DATABASE_URL="oracle-enhanced://rails:railspass@127.0.0.1:1521/systempdb" \
    ORACLE_SYSTEM_PASSWORD="threescalepass" \
    MASTER_PASSWORD="p" \
    USER_PASSWORD="p" \
    LC_ALL="en_US.UTF-8" \
    DNSMASQ="#"


RUN ./script/oracle/install-instantclient-packages.sh
RUN source /opt/app-root/etc/scl_enable
RUN DB=oracle DATABASE_URL="oracle-enhanced://rails:railspass@127.0.0.1:1521/systempdb" \
    bundle install --local --deployment --jobs $(grep -c processor /proc/cpuinfo) --retry=5

USER 1001
