Usage
=====

Installation
------------

Use Composer_:

.. code-block:: bash

    composer require cmsilex/cmsilex "dev-master"
    
Create a public directory.

Create a cmsilex directory inside your public directory ``/public/cmsilex`` that will store the
assets for the backend of cmsilex.

Copy the contents of ``/vendor/cmsilex/cmsilex/resources/public`` to the new ``/public/cmsilex`` directory.

An example nginx webserver config:

.. code:: ini

    server {
        listen 80;
        server_name leighmurray.com www.leighmurray.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name leighmurray.com www.leighmurray.com;
        root /var/www/leighmurray.com/public;

        location / {
            # try to serve file directly, fallback to front controller
            try_files $uri /index.php$is_args$args;
        }

        location ~ ^/index\.php(/|$) {
            # the ubuntu default
            fastcgi_pass   unix:/var/run/php5-fpm.sock;
            # for running on centos
            #fastcgi_pass   unix:/var/run/php-fpm/www.sock;

            fastcgi_split_path_info ^(.+\.php)(/.*)$;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param HTTPS on;

            # Prevents URIs that include the front controller. This will 404:
            # http://domain.tld/index.php/some-path
            # Enable the internal directive to disable URIs like this
            # internal;
        }

        #return 404 for all php files as we do have a front controller
        location ~ \.php$ {
            return 404;
        }

        ssl_certificate /etc/letsencrypt/live/leighmurray.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/leighmurray.com/privkey.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-R$

        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparams.pem;

        error_log /var/log/nginx/leighmurray_error.log;
        access_log /var/log/nginx/leighmurray_access.log;
    }


Bootstrap
---------

To bootstrap CMSilex, you need to require the ``vendor/autoload.php``
file and create an instance of ``CMSilex\Application``. After your controller
definitions, call the ``run`` method on your application::

    // public/index.php
    require_once __DIR__.'/../vendor/autoload.php';

    $app = new Silex\Application();

    // ... definitions

    $app->run();

Config
------

CMSilex uses YAML for its config file.

There are two settings the config needs in order to function:

.. code:: yaml

    theme: mythemedir
    db:
      driver: pdo_mysql
      dbname: mydbname
      host: 127.0.0.1
      user: mydbuser
      password: mydbpassword


  Create a file ``/config/config.yml``

Database
--------

You need to set up a database for your cms.