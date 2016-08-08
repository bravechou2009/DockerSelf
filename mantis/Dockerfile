FROM hucaeah0.cn.hpecorp.net:16688/deco_centos_7_systemd:V1.0

MAINTAINER bravechou2009@gmail.com

# This dockerfile is suitable for installing MantisBT
# for production use. The script will also look for /data volume
# where it would move permanent files MantisBT creates (attachments)

ENV http_proxy="http://web-proxy:8080" https_proxy="http://web-proxy:8080" container="docker" TERM="xterm"

LABEL version="V1.0"


# -----------------------------------------------------------------------------
# ENV Pre-configuration
# -----------------------------------------------------------------------------
ENV	MANTISBT_VERSION="1.3.0" \
	MANTISBT_APP_PATH="/app"


# -----------------------------------------------------------------------------
# Copy the required script files
# -----------------------------------------------------------------------------

COPY ./scripts/ /tmp/scripts/
COPY ./vendor/ /tmp/vendor/
RUN cp /tmp/scripts/*.sh /root/


# -----------------------------------------------------------------------------
# Base Apache, PHP , PHP-MYSQL
# -----------------------------------------------------------------------------

RUN yum clean all && \
	yum update -y && \
    yum install epel-release -y && \
    yum install -y \
        gcc make perl perl-core openssl openssl-libs openssl-devel \
        net-tools sudo multitail zip tar bzip2 patch git \
        curl curl-devel \
        php php-mbstring php-pdo php-mysql php-pgsql php-gd php-xml \
        php-soap php-Smarty re2c libxml2-devel bison bison-devel \
        libjpeg-devel libpng-devel readline-devel libtidy-devel \
        libxslt-devel libmcrypt libmcrypt-devel php-mcrypt \
        perl-libwww-perl perl-Crypt-SSLeay \
        perl-DBI perl-DBD-MySQL perl-DBD-SQLite perl-DBD-CSV perl-DBD-ODBC perl-DBD-Pg \
        ImageMagick ImageMagick-perl \
        postfix \
        mysql mysql-devel \
        httpd httpd-devel mod_php mod_ssl



# -----------------------------------------------------------------------------
# Global MantisBT configuration
# -----------------------------------------------------------------------------

RUN mkdir -p -m 750 ${MANTISBT_APP_PATH} \
	&& rm -fr /var/www/html \
	&& ln -s ${MANTISBT_APP_PATH} /var/www/html	\
	&& cd /tmp/vendor \
	&& tar -xzvf ./mantisbt.tar.gz -C ${MANTISBT_APP_PATH} \
	&& chown -R apache:apache ${MANTISBT_APP_PATH} \
	&& cp /tmp/vendor/config_inc.php -rf ${MANTISBT_APP_PATH}/mantisbt/




# -----------------------------------------------------------------------------
# Global Apache configuration changes
# -----------------------------------------------------------------------------
RUN sed -i \
	-e 's~^ServerSignature On$~ServerSignature Off~g' \
	-e 's~^ServerTokens OS$~ServerTokens Prod~g' \
	-e 's~^NameVirtualHost \(.*\)$~#NameVirtualHost \1~g' \
	/etc/httpd/conf/httpd.conf


# -----------------------------------------------------------------------------
# Disable Apache directory indexes
# -----------------------------------------------------------------------------
RUN sed -i \
	-e 's~^IndexOptions \(.*\)$~#IndexOptions \1~g' \
	-e 's~^IndexIgnore \(.*\)$~#IndexIgnore \1~g' \
	-e 's~^AddIconByEncoding \(.*\)$~#AddIconByEncoding \1~g' \
	-e 's~^AddIconByType \(.*\)$~#AddIconByType \1~g' \
	-e 's~^AddIcon \(.*\)$~#AddIcon \1~g' \
	-e 's~^DefaultIcon \(.*\)$~#DefaultIcon \1~g' \
	-e 's~^ReadmeName \(.*\)$~#ReadmeName \1~g' \
	-e 's~^HeaderName \(.*\)$~#HeaderName \1~g' \
	/etc/httpd/conf/httpd.conf


# -----------------------------------------------------------------------------
# Add default system users
# -----------------------------------------------------------------------------
RUN useradd -r -M -d ${MANTISBT_APP_PATH} -s /sbin/nologin app \
	&& useradd -r -M -d ${MANTISBT_APP_PATH} -s /sbin/nologin -G apache,app app-www \
	&& usermod -a -G app-www app \
	&& usermod -a -G app-www,app apache

# -----------------------------------------------------------------------------
# Disable Apache language based content negotiation
# -----------------------------------------------------------------------------
RUN sed -i \
	-e 's~^LanguagePriority \(.*\)$~#LanguagePriority \1~g' \
	-e 's~^ForceLanguagePriority \(.*\)$~#ForceLanguagePriority \1~g' \
	-e 's~^AddLanguage \(.*\)$~#AddLanguage \1~g' \
	/etc/httpd/conf/httpd.conf

# -----------------------------------------------------------------------------
# Disable all Apache modules and enable the minimum
# -----------------------------------------------------------------------------
RUN sed -i \
	-e 's~^\(LoadModule .*\)$~#\1~g' \
	-e 's~^\(#LoadModule version_module modules/mod_version.so\)$~\1\n#LoadModule reqtimeout_module modules/mod_reqtimeout.so~g' \
	-e 's~^#LoadModule mime_module ~LoadModule mime_module ~g' \
	-e 's~^#LoadModule log_config_module ~LoadModule log_config_module ~g' \
	-e 's~^#LoadModule setenvif_module ~LoadModule setenvif_module ~g' \
	-e 's~^#LoadModule status_module ~LoadModule status_module ~g' \
	-e 's~^#LoadModule authz_host_module ~LoadModule authz_host_module ~g' \
	-e 's~^#LoadModule dir_module ~LoadModule dir_module ~g' \
	-e 's~^#LoadModule alias_module ~LoadModule alias_module ~g' \
	-e 's~^#LoadModule expires_module ~LoadModule expires_module ~g' \
	-e 's~^#LoadModule deflate_module ~LoadModule deflate_module ~g' \
#	-e 's~^#LoadModule headers_module ~LoadModule headers_module ~g' \
	-e 's~^#LoadModule alias_module ~LoadModule alias_module ~g' \
	/etc/httpd/conf/httpd.conf

# -----------------------------------------------------------------------------
# Enable ServerStatus access via /_httpdstatus to local client
# -----------------------------------------------------------------------------
RUN sed -i \
	-e '/#<Location \/server-status>/,/#<\/Location>/ s~^#~~' \
	-e '/<Location \/server-status>/,/<\/Location>/ s~Allow from .example.com~Allow from localhost 127.0.0.1~' \
	/etc/httpd/conf/httpd.conf

# -----------------------------------------------------------------------------
# Disable the default SSL Virtual Host
# -----------------------------------------------------------------------------
RUN sed -i \
	-e '/<VirtualHost _default_:443>/,/#<\/VirtualHost>/ s~^~#~' \
	/etc/httpd/conf.d/ssl.conf

# -----------------------------------------------------------------------------
# Disable the SSL support by default
# -----------------------------------------------------------------------------
RUN mv /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.off \
	&& touch /etc/httpd/conf.d/ssl.conf \
	&& chmod 444 /etc/httpd/conf.d/ssl.conf

# -----------------------------------------------------------------------------
# Limit threads for the application user
# -----------------------------------------------------------------------------
RUN { \
		echo ''; \
		echo -e '@apache\tsoft\tnproc\t85'; \
		echo -e '@apache\thard\tnproc\t170'; \
	} >> /etc/security/limits.conf

# -----------------------------------------------------------------------------
# Global PHP configuration changes
# -----------------------------------------------------------------------------
RUN sed \
		-e 's~^; .*$~~' \
		-e 's~^;*$~~' \
		-e '/^$/d' \
		-e 's~^\[~\n\[~g' \
		/etc/php.ini \
		> /etc/php.d/00-php.ini.default \
	&& sed \
	-e 's~^;user_ini.filename =$~user_ini.filename =~g' \
	-e 's~^;cgi.fix_pathinfo=1$~cgi.fix_pathinfo=1~g' \
	-e 's~^;date.timezone =$~date.timezone = UTC~g' \
	/etc/php.d/00-php.ini.default \
	> /etc/php.d/00-php.ini


RUN ln -sf \
	/usr/share/zoneinfo/Asia/Shanghai \
	/etc/localtime

RUN sed -i \
	-e 's/;date\.timezone =/date.timezone = Asia\/Shanghai/' \
	/etc/php.ini

RUN sed -i \
	 -e "s/variables_order.*/variables_order = \"EGPCS\"/g" \
	/etc/php.ini



# -----------------------------------------------------------------------------
# Enable httpd systemd
# -----------------------------------------------------------------------------
RUN systemctl enable httpd.service




EXPOSE 80 443

CMD ["/root/boot.sh"]
