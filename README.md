Apache+PHP build pack for Wordpress
========================

This is a build pack bundling PHP and Apache for Heroku apps, which is perfect for deploying your Wordpress apps with ease.

Configuration
-------------

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------

    # use AMI ami-04c9306d
    apt-get -y update && apt-get -y install g++ gcc libssl-dev libpng-dev libxml2-dev libmysqlclient-dev libpq-dev libpcre3-dev php5-dev php-pear curl libcurl3 libcurl3-dev php5-curl libsasl2-dev
    
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
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-mbstring --with-mhash --enable-pcntl --enable-mysqlnd --with-pear --with-mysqli
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

Couchbase
---------

  curl -L https://github.com/downloads/libevent/libevent/libevent-2.0.17-stable.tar.gz -o /tmp/libevent-2.0.17-stable.tar.gz
  tar -C /tmp -xzvf /tmp/libevent-2.0.17-stable.tar.gz 
  cd /tmp/libevent-2.0.17-stable
  ./configure --prefix=/app/local
  make && make install

  curl -L http://packages.couchbase.com/clients/c/libvbucket-1.8.0.3.tar.gz -o /tmp/libvbucket-1.8.0.3.tar.gz
  tar -C /tmp -xzvf /tmp/libvbucket-1.8.0.3.tar.gz
  cd /tmp/libvbucket-1.8.0.3
  ./configure --prefix=/app/local
  make && make install

  curl -L http://packages.couchbase.com/clients/c/libcouchbase-1.0.2.tar.gz -o /tmp/libcouchbase-1.0.2.tar.gz
  tar -C /tmp -xzvf /tmp/libcouchbase-1.0.2.tar.gz
  cd /tmp/libcouchbase-1.0.2
  LDFLAGS="-L/app/local/lib" CPPFLAGS="-I/app/local/include" ./configure --prefix=/app/local --disable-couchbasemock
  make && make install

  curl -L https://github.com/couchbase/php-ext-couchbase/tarball/master -o /tmp/php-ext-couchbase.tar.gz
  tar -C /tmp -xzvf /tmp/php-ext-couchbase.tar.gz
  cd /tmp/couchbase-php-ext-couchbase-*
  /app/php/bin/phpize
  ./configure --prefix=/app/php --with-php-config=/app/php/bin/php-config --with-couchbase=/app/local/lib



Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
