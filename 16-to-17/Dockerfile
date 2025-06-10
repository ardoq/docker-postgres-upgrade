#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.#

FROM postgres:16-bullseye AS pgvector-builder-16

RUN <<EOF
apt-get update
apt-get install -y --no-install-recommends git build-essential postgresql-server-dev-16
apt-get install --reinstall ca-certificates
EOF
WORKDIR /home
RUN git clone --branch v0.7.0 https://github.com/pgvector/pgvector.git
WORKDIR /home/pgvector
RUN make
RUN make install

FROM postgres:17-bullseye AS pgvector-builder-17

RUN <<EOF
apt-get update
apt-get install -y --no-install-recommends git build-essential postgresql-server-dev-17
apt-get install --reinstall ca-certificates
EOF
WORKDIR /home
RUN git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git
WORKDIR /home/pgvector
RUN make
RUN make install


FROM postgres:17-bullseye

RUN sed -i 's/$/ 16/' /etc/apt/sources.list.d/pgdg.list

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		postgresql-16='16.9-1.pgdg110+1' \
	; \
	rm -rf /var/lib/apt/lists/*

ENV PGBINOLD /usr/lib/postgresql/16/bin
ENV PGBINNEW /usr/lib/postgresql/17/bin

ENV PGDATAOLD /var/lib/postgresql/16/data
ENV PGDATANEW /var/lib/postgresql/17/data


COPY --from=pgvector-builder-16 /usr/lib/postgresql/16/lib/bitcode/vector.index.bc /usr/lib/postgresql/16/lib/bitcode/vector.index.bc
COPY --from=pgvector-builder-16 /usr/lib/postgresql/16/lib/vector.so /usr/lib/postgresql/16/lib/vector.so
COPY --from=pgvector-builder-16 /usr/share/postgresql/16/extension/ /usr/share/postgresql/16/extension/

COPY --from=pgvector-builder-17 /usr/lib/postgresql/17/lib/bitcode/vector.index.bc /usr/lib/postgresql/17/lib/bitcode/vector.index.bc
COPY --from=pgvector-builder-17 /usr/lib/postgresql/17/lib/vector.so /usr/lib/postgresql/17/lib/vector.so
COPY --from=pgvector-builder-17 /usr/share/postgresql/17/extension/ /usr/share/postgresql/17/extension/


RUN set -eux; \
	mkdir -p "$PGDATAOLD" "$PGDATANEW"; \
	chown -R postgres:postgres /var/lib/postgresql

WORKDIR /var/lib/postgresql

COPY docker-upgrade /usr/local/bin/

ENTRYPOINT ["docker-upgrade"]

# recommended: --link
CMD ["pg_upgrade"]
