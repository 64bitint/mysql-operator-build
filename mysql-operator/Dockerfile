
ARG MYSQL_REPO_URL="http://repo.mysql.com"
ARG MYSQL_OPERATOR_PYTHON_DEPS_VERSION="3.9.5"
ARG MYSQL_OPERATOR_VERSION=2.0.7
ARG MYSQL_SHELL_VERSION=8.0.31
ARG MYSQL_CONFIG_PKG="mysql80-community-release"
ARG MYSQL_SHELL_REPO="mysql-tools-community"

FROM oraclelinux:8 AS pip-stage


RUN dnf install -y python39
RUN dnf install -y python39-pip

COPY requirements.txt .

RUN pip3 install --target=/tmp/site-packages -r requirements.txt

FROM oraclelinux:8-slim AS mysql-operator
ARG MYSQL_REPO_URL
ARG MYSQL_SHELL_VERSION
ARG MYSQL_OPERATOR_VERSION
ARG MYSQL_CONFIG_PKG
ARG MYSQL_SHELL_REPO

COPY --from=pip-stage /tmp/site-packages /usr/lib/mysqlsh/python-packages

RUN rpm -U $MYSQL_REPO_URL/$MYSQL_CONFIG_PKG-el8.rpm \
  && microdnf update && echo "[main]" > /etc/dnf/dnf.conf \
  && microdnf install --enablerepo=$MYSQL_SHELL_REPO -y mysql-shell-$MYSQL_SHELL_VERSION \
  && microdnf remove $MYSQL_CONFIG_PKG \
  && microdnf clean all

RUN groupadd -g27 mysql && useradd -u27 -g27 mysql

RUN mkdir /mysqlsh && chown 2 /mysqlsh

COPY --from=mysql/mysql-operator:$MYSQL_SHELL_VERSION-$MYSQL_OPERATOR_VERSION /usr/lib/mysqlsh/python-packages/mysqloperator /usr/lib/mysqlsh/python-packages/mysqloperator

# Workaround for BC issue with newest Python library for Kubernetes
# See move here: https://github.com/kubernetes-client/python/issues/1718
RUN sed -i "s/available_replicas=None,/available_replicas=0,/" /usr/lib/mysqlsh/python-packages/kubernetes/client/models/v1_stateful_set_status.py


USER 2

ENV HOME=/mysqlsh