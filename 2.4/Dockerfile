FROM httpd:alpine

# These variables are inherited from the httpd:alpine image:
# ENV HTTPD_PREFIX /usr/local/apache2
# WORKDIR "$HTTPD_PREFIX"

# Copy in our configuration files.
COPY conf/ conf/

RUN set -ex; \
    #Install usermod
    echo http://dl-2.alpinelinux.org/alpine/edge/community/ >> /etc/apk/repositories; \
    apk --no-cache add shadow; \
    #
    # if uid not specified then use default uid for user nobody
    if [[ -z "${PUID}" ]]; then \
     PUID="99"; \
    fi; \
    #
    # if gid not specifed then use default gid for group users
    if [[ -z "${PGID}" ]]; then \
     PGID="100"; \
    fi; \
    #
    # set user nobody to specified user id (non unique)
    usermod -o -u "${PUID}" nobody; \
    echo "[info] Env var PUID  defined as ${PUID}"; \
    #
    # set group users to specified group id (non unique)
    groupmod -o -g "${PGID}" users; \
    echo "[info] Env var PGID defined as ${PGID}"; \
    #
    # Create empty default DocumentRoot.
    mkdir -p "/var/www/html"; \
    # Create directories for Dav data and lock database.
    mkdir -p "/var/lib/dav/data"; \
    touch "/var/lib/dav/DavLock"; \
    chown -R "${PUID}":"${PGID}" "/var/lib/dav"; \
    #
    # Enable DAV modules.
    for i in dav dav_fs; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    #
    # Make sure authentication modules are enabled.
    for i in authn_core authn_file authz_core authz_user auth_basic auth_digest; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    #
    # Make sure other modules are enabled.
    for i in alias headers mime setenvif; do \
        sed -i -e "/^#LoadModule ${i}_module.*/s/^#//" "conf/httpd.conf"; \
    done; \
    #
    # Run httpd as "www-data" (instead of "daemon").
    #for i in User Group; do \
    #    sed -i -e "s|^$i .*|$i www-data|" "conf/httpd.conf"; \
    #done; \
    sed -i -e "s|^User .*|User ${PUID}|" "conf/httpd.conf"; \
    sed -i -e "s|^Group .*|Group ${PGID}|" "conf/httpd.conf"; \
    #
    # Include enabled configs and sites.
    printf '%s\n' "Include conf/conf-enabled/*.conf" \
        >> "conf/httpd.conf"; \
    printf '%s\n' "Include conf/sites-enabled/*.conf" \
        >> "conf/httpd.conf"; \
    #
    # Enable dav and default site.
    mkdir -p "conf/conf-enabled"; \
    mkdir -p "conf/sites-enabled"; \
    ln -s ../conf-available/dav.conf "conf/conf-enabled"; \
    ln -s ../sites-available/default.conf "conf/sites-enabled"; \
    # Install openssl if we need to generate a self-signed certificate.
    apk add --no-cache openssl

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
EXPOSE 80/tcp 443/tcp
ENTRYPOINT [ "docker-entrypoint.sh" ]
CMD [ "httpd-foreground" ]
