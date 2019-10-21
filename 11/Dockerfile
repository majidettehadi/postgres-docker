FROM majid7221/debian:stretch

# explicitly set user/group IDs
RUN set -ex \
    && groupadd -r postgres --gid=999 \
    && useradd -r -g postgres --uid=999 postgres

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN set -ex \
    && touch /usr/share/locale/locale.alias \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        locales \
    && rm -rf /var/lib/{apt,dpkg,cache,log}/ /tmp/* /var/tmp/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN mkdir /docker-entrypoint-initdb.d

RUN set -ex \
	&& key='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" \
	&& gpg --export "$key" > /etc/apt/trusted.gpg.d/postgres.gpg \
	&& rm -rf "$GNUPGHOME"

ENV PG_MAJOR 11
RUN set -ex \
	&& echo "deb http://apt.postgresql.org/pub/repos/apt/ stretch-pgdg main $PG_MAJOR" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
	&& apt-get install -y \
        postgresql-common \
	&& sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
	&& apt-get install -y \
		"postgresql-$PG_MAJOR" \
	&& rm -rf /var/lib/{apt,dpkg,cache,log}/ /tmp/* /var/tmp/* 

# make the sample config easier to munge (and "correct by default")
RUN set -ex \
    && mv -v "/usr/share/postgresql/$PG_MAJOR/postgresql.conf.sample" /usr/share/postgresql/ \
	&& ln -sv ../postgresql.conf.sample "/usr/share/postgresql/$PG_MAJOR/" \
	&& sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/share/postgresql/postgresql.conf.sample \
    && mkdir -p /var/run/postgresql \
    && chown -R postgres:postgres /var/run/postgresql \
    && chmod 2777 /var/run/postgresql

ENV PATH $PATH:/usr/lib/postgresql/$PG_MAJOR/bin
ENV PGDATA /var/lib/postgresql/data
RUN set -ex \
    && mkdir -p "$PGDATA" \
    && chown -R postgres:postgres "$PGDATA" \
    && chmod 777 "$PGDATA"
    
VOLUME ["/var/lib/postgresql/data"]
VOLUME ["/etc/postgresql/conf.d"]

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
