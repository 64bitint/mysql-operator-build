ARG MYSQL_SHELL_VERSION=8.0.31

FROM oraclelinux:8-slim
ARG MYSQL_SHELL_VERSION

ARG MYSQL_CLIENT_PACKAGE=mysql-community-client-$MYSQL_SHELL_VERSION
ARG MYSQL_ROUTER_PACKAGE=mysql-router-community-$MYSQL_SHELL_VERSION

RUN /usr/sbin/groupadd -g 999 -r mysqlrouter >/dev/null \
   && /usr/sbin/useradd -M -N -u 999 -g mysqlrouter -r -d /var/lib/mysqlrouter -s /bin/false \
   -c "MySQL Router" mysqlrouter \
   && rpm -U https://repo.mysql.com/mysql80-community-release-el8.rpm \
   && microdnf install -y --disablerepo=ol8_appstream \
   $MYSQL_CLIENT_PACKAGE $MYSQL_ROUTER_PACKAGE \
   && microdnf clean all

COPY run.sh /

HEALTHCHECK CMD mysqladmin --port 6446 --protocol TCP ping 2>&1 | grep Access || exit 1

EXPOSE 6446 6447 6448 6449 8443

USER 999:999

ENTRYPOINT ["/run.sh"]

CMD ["mysqlrouter"]