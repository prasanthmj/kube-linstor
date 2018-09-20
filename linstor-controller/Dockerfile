FROM ubuntu:xenial
ENV DEBIAN_FRONTEND noninteractive

# Install repos and system upgrade
RUN apt-get -y update \
 && apt-get install -y wget gnupg \
 && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys CC1B5A793C04BB3905AD837734893610CEAA9512 \
 && echo "deb http://ppa.launchpad.net/linbit/linbit-drbd9-stack/ubuntu xenial main" \
      > /etc/apt/sources.list.d/linbit.list \
 && apt-get update \
 && apt-get -y upgrade \
 && apt-get clean

# Install tini
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

# Install linstor-controller
RUN apt-get update \
 && apt-get install -y linstor-controller linstor-client \
 && apt-get clean \
 && mkdir -p /config /logs \
 && /usr/share/linstor-server/bin/linstor-config create-db-file \
      /data/linstordb > /config/database.cfg

# Install postgresql connector
ENV JDBC_POSTGRESQL_VERSION 42.2.2
RUN wget -O /usr/share/linstor-server/lib/postgresql-${JDBC_POSTGRESQL_VERSION}.jar \
      https://jdbc.postgresql.org/download/postgresql-${JDBC_POSTGRESQL_VERSION}.jar \
 && sed -i "/^CLASSPATH=/ s|$|:\$APP_HOME/lib/postgresql-${JDBC_POSTGRESQL_VERSION}.jar|" \
      /usr/share/linstor-server/bin/Controller

# Install mysql connector
ENV JDBC_MYSQL_VERSION 8.0.12
RUN wget -O - https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-${JDBC_MYSQL_VERSION}.tar.gz \
      | tar xzvf - mysql-connector-java-${JDBC_MYSQL_VERSION}/mysql-connector-java-${JDBC_MYSQL_VERSION}.jar \
 && mv mysql-connector-java-${JDBC_MYSQL_VERSION}/mysql-connector-java-${JDBC_MYSQL_VERSION}.jar . \
 && rmdir mysql-connector-java-${JDBC_MYSQL_VERSION} \
 && sed -i "/^CLASSPATH=/ s|$|:\$APP_HOME/lib/mysql-connector-java-${JDBC_MYSQL_VERSION}.jar|" \
      /usr/share/linstor-server/bin/Controller

CMD [ "/usr/share/linstor-server/bin/Controller", "--logs=/logs", "--config-directory=/config" ]
