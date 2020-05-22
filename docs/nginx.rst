==========================================================================================
Nginx
==========================================================================================

.. rubric:: Compile Nginx From Sources

If you want to proxy https traffic you will need to compile with ssl enabled. For that you
will need openssl which you can install with homebrew.

.. code-block:: terminal

    $ brew install openssl

Download the sources `here <http://nginx.org/en/download.html>`_, which you will than have
to build with the proper command line flags.

Nginx will look for configuration in a location that is specified at compile time. To
simplify the example we assume you will build nginx where you will run it.

.. code-block:: terminal

    $ curl -L -O http://nginx.org/download/nginx-x.y.z.tar.gz
    $ tar -xf nginx-x.y.z.tar.gz && cd nginx-x.y.z/
    $ ./configure \
        --prefix="$(dirname $(pwd))" \
        --with-http_ssl_module \
        --with-cc-opt=-I/usr/local/Cellar/openssl@1.1/1.1.1g/include \
        --with-ld-opt=-L/usr/local/Cellar/openssl@1.1/1.1.1g/lib
    $ make

You should now have a working nginx binary, but you still do not have any configuration.
your service directory (``__SERVICES__/nginx``) you can run the following. You may also
want to copy the default html directory

.. code-block:: terminal

    $ cp -r ./nginx-x.y.z/conf .
    $ cp -r ./nginx-x.y.z/html .

in your ``./run`` script you can then put the following

.. code-block:: sh

    #! /usr/bin/env sh

    ./nginx-x.y.z/objs/nginx

.. rubric:: Configuration

You may want to disable deamon mode, such that you can run the process at will and can
terminate it more easily by hitting ``ctl-c``. And you may want to switch the ports to the
range for your product.

.. code-block:: terminal

    $ sed -i -e 's/\(worker_processes.*$\)/\1\'$'\n''daemon  off;/' ./conf/nginx.conf
    $ sed -i -e 's/\(listen[[:space:]]*\)80;/\10080;/' ./conf/nginx.conf

example ssl configuration:

.. code-block:: nginx

    server {
        listen                     14443 ssl;
        server_name                some.example.com

        ssl_certificate            path_to_chain.cert;
        ssl_certificate_key        path_to_private.key;
        ssl_session_cache          shared:SSL:1m;
        ssl_session_timeout        5m;

        ssl_ciphers                HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass  http://localhost:<port>;
        }
    }

If you are using the certificate authority project, than you can create the chain and the
key as follows.

.. code-block:: terminal

    $ ./scripts/create-leaf-cert.sh -cn='www.testinc.org' -alt='testinc.org' -alt='*.testinc.org'
    $ cat ./data/main/www_testinc_org.cert ./data/main/www_testinc_org.chain > www_testinc_org.cert
    $ openssl rsa -in ./data/main/www_testinc_org.key -out www_testinc_org.key

The key will not have a password anymore (otherwise you will have to type it in every time
you start nginx. This means that anyone can read the key. You can protect it with file
permissions if you want, or provide a password anyway. It is up to you to decide what kind
of security profile you want to use in your development setups.
