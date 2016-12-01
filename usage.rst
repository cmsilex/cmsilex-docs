Usage
=====

Installation
------------

Create a ``composer.json`` file:

.. code:: json

    {
        "minimum-stability": "dev",
        "prefer-stable": true
    }

Use Composer_:

.. code:: bash

    composer require cmsilex/cmsilex "dev-master"
    
Create a public directory.

Create a ``cmsilex`` directory inside your public directory eg. ``/public/cmsilex`` that will store the
assets for the backend of cmsilex.

Copy the contents of ``/vendor/cmsilex/cmsilex/resources/public`` to the new ``/public/cmsilex`` directory.

An example nginx webserver config:

.. code:: ini

    server {
        listen 80;
        server_name yoursite.com www.yoursite.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name yoursite.com www.yoursite.com;
        root /var/www/yoursite.com/public;

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

        ssl_certificate /etc/letsencrypt/live/yoursite.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/yoursite.com/privkey.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-R$

        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/nginx/dhparams.pem;

        error_log /var/log/nginx/yoursite_error.log;
        access_log /var/log/nginx/yoursite_access.log;
    }


Bootstrap
---------

To bootstrap CMSilex, you need to require the ``vendor/autoload.php``
file and create an instance of ``CMSilex\Application``. 

Create a generic bootstrap file ``/bootstrap.php`` that can be used both by the webserver and CLI:

.. code: php

    <?php

    require_once __DIR__ . "/vendor/autoload.php";

    $app = new \CMSilex\Application();

    // register additional services eg...
    // $app->register(new \CMSilex\MoodTracker\MoodTracker());

    return $app;


In ``/public/index.php`` require your bootstrap file which returns your application and
call the ``run`` method on your application::

    <?php

    $app = require __DIR__ . "/../bootstrap.php";

    $app->run();


Config
------

CMSilex uses YAML for its config file.

Create a file ``/config/config.yml``.

.. code:: yaml

    # Turn debug on or off
    # debug: false

    # the directory within /themes where your frontend theme resides
    theme: mythemedir
    
    # Enable or disable the /register path to allow new user registration
    # register: false
    
    # example mysql db config
    db:
      driver: pdo_mysql
      dbname: mydbname
      host: 127.0.0.1
      user: mydbuser
      password: mydbpassword
    
    # example sqlite db config
    # db:
    #   driver: pdo_sqlite
    #   path: /path/to/sqlite.db


    
CLI-Config
==========

In order for doctrine command line to work you need a php config file at ``/config/cli-config.php``:

.. code:: php

    <?php
    use Doctrine\ORM\Tools\Console\ConsoleRunner;

    // replace with file to your own project bootstrap
    $app = require_once __DIR__ . "/../bootstrap.php";

    return ConsoleRunner::createHelperSet($app['em']);

    

Database
--------

You need to set up a database for your cms.

.. code:: bash

    vendor/bin/doctrine orm:schema:create
    