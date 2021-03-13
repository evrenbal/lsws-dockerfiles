FROM ubuntu:18.04
ENV LS_FD='/usr/local/lsws'

ARG LSWS_VERSION
ARG PHP_VERSION

RUN apt-get update && apt-get install wget curl tzdata cron -y
COPY lsws-install.sh /
RUN chmod +x /lsws-install.sh && bash /lsws-install.sh $LSWS_VERSION

RUN apt-get install mysql-client $PHP_VERSION $PHP_VERSION-common $PHP_VERSION-dev $PHP_VERSION-mysql $PHP_VERSION-opcache $PHP_VERSION-apcu \
    $PHP_VERSION-curl $PHP_VERSION-json $PHP_VERSION-pear $PHP_VERSION-imagick $PHP_VERSION-igbinary $PHP_VERSION-redis $PHP_VERSION-memcached $PHP_VERSION-intl -y

RUN wget -O -  https://get.acme.sh | sh

EXPOSE 7080

ADD docker.xml $LS_FD/conf/templates/docker.xml
ADD setup_docker.sh $LS_FD/bin/setup_docker.sh
ADD httpd_config.conf $LS_FD/conf/httpd_config.conf
ADD htpasswd $LS_FD/admin/conf/htpasswd

#INSTALL PECL EXTENSIONS
RUN	apt-get update \
    &&/usr/local/lsws/$PHP_VERSION/bin/pecl channel-update pecl.php.net \
  && cd ~ \
	&& /usr/local/lsws/$PHP_VERSION/bin/pecl pecl install amqp \
	&& printf "yes\n" | /usr/local/lsws/$PHP_VERSION/bin/pecl install swoole \
	&& /usr/local/lsws/$PHP_VERSION/bin/pecl pecl install mongodb \
	&& /usr/local/lsws/$PHP_VERSION/bin/pecl clear-cache

	
RUN cd /usr/local/lsws/$PHP_VERSION/bin/ \
	&& echo "extension=swoole.so" >> php.ini \
	&& echo "extension=amqp.so" >> php.ini \
	&& echo "extension="mongodb.so" >> php.ini

RUN $LS_FD/bin/setup_docker.sh $PHP_VERSION && rm $LS_FD/bin/setup_docker.sh
RUN chown 999:999 $LS_FD/conf -R
RUN cp -RP $LS_FD/conf/ $LS_FD/.conf/
RUN cp -RP $LS_FD/admin/conf $LS_FD/admin/.conf/
RUN ln -sf $LS_FD/$PHP_VERSION/bin/lsphp $LS_FD/fcgi-bin/lsphp7
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /var/www/vhosts/
