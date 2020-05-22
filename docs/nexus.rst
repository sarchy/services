==========================================================================================
Nexus
==========================================================================================

.. code-block:: terminal

    $ mkdir nexus && cd nexus;
    $ curl -L -O http://download.sonatype.com/nexus/3/nexus-3.23.0-03-unix.tar.gz
    $ tar -xf nexus-x.y.z-unix.tar.gz .
    $ ln -s nexus-x.y.z current

.. code-block:: terminal

    $ sed -i current/etc/nexus-default.properties \
        -e 's/application-port=.*$/application-port=14082'


.. code-block:: terminal

    $ ./current/bin/nexus run


    server {
        listen                     14443 ssl;
        server_name                nexus.testinc.org;

        ssl_certificate            www_testinc_org.cert;
        ssl_certificate_key        www_testinc_org.key;
        ssl_session_cache          shared:SSL:1m;
        ssl_session_timeout        5m;

        ssl_ciphers                HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass  http://localhost:14082;
            proxy_set_header Host $host:$server_port;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto 'https';
        }
    }
