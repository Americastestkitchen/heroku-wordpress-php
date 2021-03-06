Apache+PHP buildpack for Wordpress
===================================

This is a buildpack bundling PHP and Apache for Heroku, which is perfect to use for Wordpress apps such as [The Feed](http://www.americastestkitchenfeed.com)

See [The Feed Repo](https://github.com/Americastestkitchen/feed) for more information on how its .buildpack file loads the heroku-wordpress-php buildpack.


-------------
Configuration

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini


----------------------
Pre-compiling binaries

Necessary only if you are looking to compile and host your own binaries. After you complete these steps, you will have to upload them to S3 and then point bin/compile to the new sources.

    # use AMI ami-04c9306d
    apt-get -y update && apt-get -y install g++ gcc libssl-dev libpng-dev libjpeg-dev libxml2-dev libmysqlclient-dev libpq-dev libpcre3-dev php5-dev php-pear curl libcurl3 libcurl3-dev php5-curl libsasl2-dev

    #download all the srcs
    curl -L http://www.apache.org/dist/httpd/httpd-2.2.22.tar.gz -o /tmp/httpd-2.2.22.tar.gz
    curl -L http://us.php.net/get/php-5.3.10.tar.gz/from/us2.php.net/mirror -o /tmp/php-5.3.10.tar.gz
    curl -L https://launchpad.net/libmemcached/1.0/1.0.4/+download/libmemcached-1.0.4.tar.gz -o /tmp/libmemcached-1.0.4.tar.gz
    curl -L http://pecl.php.net/get/memcached-2.0.1.tgz -o /tmp/memcached-2.0.1.tgz

    #untar all the srcs
    tar -C /tmp -xzvf /tmp/httpd-2.2.22.tar.gz
    tar -C /tmp -xzvf /tmp/php-5.3.10.tar.gz
    tar -C /tmp -xzvf /tmp/libmemcached-1.0.4.tar.gz
    tar -C /tmp -xzvf /tmp/memcached-2.0.1.tgz

    #make the directories
    mkdir /app
    mkdir /app/{apache,php,local}
    mkdir /app/php/ext
    mkdir /app/local/lib

    #copy libs
    cp /usr/lib/libmysqlclient* /app/local/lib/
    cp /usr/lib/libsasl2* /app/local/lib/


    # apache
    cd /tmp/httpd-2.2.22
    ./configure --prefix=/app/apache --enable-rewrite --enable-so --enable-deflate --enable-expires --enable-headers
    make && make install

    # php
    cd /tmp/php-5.3.10
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-mbstring --with-mhash --enable-pcntl --enable-mysqlnd --with-pear --with-mysqli --with-jpeg-dir=/usr/lib
    make && make install

    # libmemcached
    cd /tmp/libmemcached-1.0.4
    ./configure --prefix=/app/local
    make && make install

    # pecl memcached
    cd /tmp/memcached-2.0.1
    # edit config.m4 line 21 so no => yes ############### IMPORTANT!!! ###############
    sed -i -e '21 s/no, no/yes, yes/' /tmp/memcached-2.0.1/config.m4
    /app/php/bin/phpize
    ./configure --with-libmemcached-dir=/app/local/ --prefix=/app/php --with-php-config=/app/php/bin/php-config
    make && make install

    /app/php/bin/pear config-set php_dir /app/php
    /app/php/bin/pecl install memcache
    /app/php/bin/pecl install apc

    # make it a little leaner
    rm -rf /app/apache/manual/

    # package
    cd /app
    echo '2.2.22' > apache/VERSION
    tar -zcvf apache-2.2.22.tar.gz apache
    echo '5.3.10' > php/VERSION
    tar -zcvf php-5.3.10.tar.gz php local

-------
Hacking

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


----
Meta

Created by Pedro Belo, modified by ATK.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
