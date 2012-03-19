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

    # use AMI ami-04c9306d and ssh ubuntu@
    sudo apt-get -y update && sudo apt-get -y install g++ gcc libssl-dev libpng-dev libxml2-dev libmysqlclient-dev libpq-dev libpcre3-dev php5-dev php-pear curl libcurl3 libcurl3-dev php5-curl libsasl2-dev
    
    #download all the srcs
    curl -L http://www.apache.org/dist/httpd/httpd-2.2.22.tar.gz -o /tmp/httpd-2.2.22.tar.gz
    curl -L http://us.php.net/get/php-5.3.10.tar.gz/from/us2.php.net/mirror -o /tmp/php-5.3.10.tar.gz
    curl -L http://pecl.php.net/get/memcache-2.2.6.tgz -o /tmp/memcache-2.2.6.tgz
    
    #untar all the srcs
    tar -C /tmp -xzvf /tmp/httpd-2.2.22.tar.gz
    tar -C /tmp -xzvf /tmp/php-5.3.10.tar.gz
    tar -C /tmp -xzvf /tmp/libmemcached-1.0.4.tar.gz
    tar -C /tmp -xzvf /tmp/memcache-2.2.6.tgz
    
    #make the directories
    sudo mkdir /app
    sudo mkdir /app/{apache,php,local}
    sudo mkdir /app/php/ext
    sudo mkdir /app/local/lib
    
    #copy libs
    cp /usr/lib/libmysqlclient* /app/local/lib/
    cp /usr/lib/libsasl2* /app/local/lib/
    
    
    # apache
    cd /tmp/httpd-2.2.22
    ./configure --prefix=/app/apache --enable-rewrite --enable-so --enable-deflate --enable-expires --enable-headers
    make
    sudo make install
    
    # php
    cd /tmp/php-5.3.10
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-mbstring --with-mhash --enable-pcntl --enable-mysqlnd --with-pear --with-mysqli
    make
    sudo make install
    
    # pecl memcache (NOT memcached) -- not sure this is necessary
    cd /tmp/memcache-2.2.6
    /app/php/bin/phpize
    ./configure --prefix=/app/php --with-php-config=/app/php/bin/php-config
    make
    sudo make install
    
    /app/php/bin/pear config-set php_dir /app/php
    sudo /app/php/bin/pecl install memcache
    sudo /app/php/bin/pecl install apc
    
    # make it a little leaner
    rm -rf /app/apache/manual/
     
    # package -- I had to use vi to do this, as I was getting permission denied on the echo's
    cd /app
    echo '2.2.22' > apache/VERSION
    sudo tar -zcvf apache-2.2.22.tar.gz apache
    echo '5.3.10' > php/VERSION
    sudo tar -zcvf php-5.3.10.tar.gz php local

Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo. Modified by ATK.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
