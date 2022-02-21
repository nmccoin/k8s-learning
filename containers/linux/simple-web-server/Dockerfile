# The image we start with
FROM httpd:2.4-alpine

# export correct path for httpd-foreground to execute after copying during container build
ENV PATH="/usr/local/apache2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

#copy our cheesy little website
COPY public-html /usr/local/apache2/htdocs/

#add our modified httpd.conf file to listen on port 9000
ADD my-httpd.conf /usr/local/apache2/conf/httpd.conf

#Custom changes
RUN apk update && apk upgrade
RUN apk -q add curl vim libcap

# change uid and gid for www-data user - inside PSJH these need to be between 100-1100
RUN apk --no-cache add shadow && \
    usermod -u 150 www-data && \
    groupmod -g 150 www-data

#Change access righs to conf, logs, bin from root to www-data
RUN chown -hR www-data:www-data /usr/local/apache2/

#setcap to bind to privileged ports as non-root
RUN setcap 'cap_net_bind_service=+ep' /usr/local/apache2/bin/httpd
RUN getcap /usr/local/apache2/bin/httpd

HEALTHCHECK --interval=60s --timeout=30s CMD nc -zv localhost 80 || exit 1
#Run as a www-data
USER www-data