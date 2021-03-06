#PHP Images can be found at https://hub.docker.com/_/php/
FROM php:7.3-alpine3.12
ENV WP_VERSION=latest

#Download and unzip newest version of wordpress
WORKDIR /var/www/localhost/htdocs
RUN echo "Download and unzip newest version of wordpress" \
    && wget https://wordpress.org/$WP_VERSION.tar.gz \
    && tar -xzf latest.tar.gz \
    && rm latest.tar.gz \
    && cp wordpress/wp-config-sample.php wordpress/wp-config.php \
#Get salt values from wordpress and insert them into wp-config.php
    && echo "Configure salts" \
    && salts=$(curl https://api.wordpress.org/secret-key/1.1/salt/) \
    && saltsEscapedForSed=${salts//$'\n'/\\$'\n'} \
    && sed -i "/NONCE_SALT/a $saltsEscapedForSed" wordpress/wp-config.php \
    && sed -i '/put your unique phrase here/d' wordpress/wp-config.php \
#Move Wordpress to root of web dir
    && mv wordpress/* ./ \
    && rm -r wordpress

#Copy entrypoint script to file system
COPY docker-entrypoint.sh /usr/bin/docker-entrypoint.sh

#Cleanup unneeded user
RUN deluser www-data

#COPY app ./

#Copy .htaccess
COPY .htaccess /var/www/localhost/htdocs/.htaccess

#Concatenated RUN commands
#php-xml required for Amazon Web Services SDK used by WP Offload Media Lite
RUN apk add --update apache2 php7-apache2 php7-mbstring php7-session php7-json php7-pdo php7-openssl php7-common php7-mysqli \
        php7-tokenizer php7-pdo php7-pdo_mysql php7-xml php7-simplexml php7-iconv php7-gd php7-curl php7-imap php7-cgi fcgi \
        php7-soap php7-xmlrpc php7-posix php7-mcrypt php7-gettext php7-ldap php7-ctype php7-dom php-xml php7-bcmath php7-exif php7-fileinfo php7-pecl-imagick\
    && sed -i '/LoadModule rewrite_module/s/^#//g' /etc/apache2/httpd.conf \
    && sed -i '/LoadModule session_module/s/^#//g' /etc/apache2/httpd.conf \
    && sed -i 's/AllowOverride\ None/AllowOverride\ All/g' /etc/apache2/httpd.conf \
    && docker-php-ext-install pdo_mysql \
    && rm  -rf /tmp/* /var/cache/apk/*

#Write apache logs to console - https://serverfault.com/questions/711168/writing-apache2-logs-to-stdout-stderr
RUN ln -sf /proc/self/fd/1 /var/log/apache2/access.log && \
    ln -sf /proc/self/fd/1 /var/log/apache2/error.log

#Delete unneeded leftover index.html
RUN rm /var/www/localhost/htdocs/index.html

#Set file permissions
RUN echo "Setting file permissions" \
    && chown -R apache:apache /var/www/localhost/htdocs \
    && chmod 775 /var/www/localhost/htdocs \
    && find /var/www/localhost/htdocs -type d -exec chmod 775 {} \; \
    && find /var/www/localhost/htdocs -type f -exec chmod 664 {} \; \
    && ls -l /var/www/localhost/htdocs/

#Launch the httpd in foreground
CMD ["/usr/bin/docker-entrypoint.sh"]