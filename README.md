Apache+PHP build pack
========================

This is a build pack bundling PHP and Apache for Heroku apps.

Configuration
-------------

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------

    apt-get -y update
    apt-get -y install g++ gcc
    apt-get -y install libssl-dev libpng-dev libxml2-dev libmysqlclient-dev libpq-dev libpcre3-dev php5-dev php-pear curl libcurl3 libcurl3-dev php5-curl

    # apache
    mkdir /app
    curl -L http://www.apache.org/dist/httpd/httpd-2.2.22.tar.gz -o /tmp/httpd-2.2.22.tar.gz
    tar -C /tmp -xzvf /tmp/httpd-2.2.22.tar.gz
    cd /tmp/httpd-2.2.22
    ./configure --prefix=/app/apache --enable-rewrite --enable-so --enable-deflate --enable-expires --enable-headers
    make
    make install
    cd ..
    
    # php
    curl -L http://us.php.net/get/php-5.3.10.tar.gz/from/us2.php.net/mirror -o /tmp/php.tar.gz
    tar -C /tmp -xzvf /tmp/php.tar.gz
    cd /tmp/php-5.3.10/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-mbstring --with-mhash --enable-pcntl --enable-mysqlnd --with-pear --with-mysqli
    make
    make install
    cd ..

    # extensions and libraries
    mkdir /app/local
    curl -L https://launchpad.net/libmemcached/1.0/1.0.4/+download/libmemcached-1.0.4.tar.gz -o /tmp/libmemcached-1.0.4.tar.gz
    cd /tmp
    tar -xzvf libmemcached-1.0.4.tar.gz
    cd libmemcached-1.0.4
    ./configure --prefix=/app/local
    make
    make install
    cd /tmp
    curl -L -O http://pecl.php.net/get/memcached-2.0.1.tgz
    tar -xzvf memcached-2.0.1.tgz
    cd memcached-2.0.1
    /app/php/bin/phpize
    ./configure --with-libmemcached-dir=/app/local/ --prefix=/app/php --with-php-config=/app/php/bin/php-config
    make
    make install
    
    /app/php/bin/pear config-set php_dir /app/php
    /app/php/bin/pecl install apc
    /app/php/bin/pecl install memcache

    mkdir /app/local/lib
    cp /usr/lib/libmysqlclient* /app/local/lib/

    # php extensions
    mkdir /app/php/ext
    
    # package
    cd /app
    echo '2.2.22' > apache/VERSION
    tar -zcvf apache-2.2.22.tar.gz apache
    echo '5.3.10' > php/VERSION
    tar -zcvf php-5.3.10.tar.gz php local


Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
