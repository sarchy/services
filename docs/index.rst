==========================================================================================
Services
==========================================================================================

Create a space that can function as a deploy target. You may have a bunch of repositories
that together combine to create a product. All these repositories are likely checked out
in a folder.

.. code-block:: directory

    ./workspace-product
      |- __SERVICES__/
      |- git-repo-one/
      |- git-repo-two/
      `- git-repo-three/

The idea is to collect all the required services under ``__SERVICES__`` and preconfigure
them such that you can just run them preferably by having a ``./run`` script in each of
them.

These services may be things like a database (postgres, mysql, mongo, neo4j, ...), brokers
(activemq, kafka, rabbitmq, ...) or supporting functionality (nginx, nexus, kibana,
jenkins, ...)

Structure Of A Service
==========================================================================================

Every service can be have a directory directly under ``./__SERVICES__``. In that directory
you would have a ``current`` symlink to your currently used version, perhaps a data
directory, a config directory, and a run script.

.. code-block:: directory

    ./__SERVICES__/
      |- service-one/
      |    |- data/
      |    |- conf/
      |    |- extracted-service-x.y.z/
      |    |- current -> extracted-service-x.y.z
      |    `- run*
      `- service-two/
           |- data/
           |- conf/
           |- extracted-service-x.y.z/
           |- current -> extracted-service-x.y.z
           `- run*


.. rubric:: services

The instruction in for the services in question will be given with mac in mind. They can
still serve as an illustration on how it would function on other platforms, but the
specificts are most often given for mac.

.. toctree::
    :maxdepth: 1

    ./nginx
    ./nexus


Using Signed Certificates
==========================================================================================

We assume that you will be using the certificate authority project. This lets you create
certificates signed by a self signed root or intermediate certificates. In order to use
them without getting warnings all over the place you will need to trust the root
certificate or at least the intermediate certificate that is currently being used.

On mac you can trust it by just opening it which lets you add it to the keychain. Once
added you need to find it again open the details view by double clicking and change the
trust settings for the certificate to always trust.

This will let browsers and some other services trust any certificate generated with your
own root.

Java processes however will still not trust your certificate. You will have to add it to
the cacerts of your java installation. For every jdk/jre that you use you will have to do
the following

.. code-block:: terminal

    $ keytool -import \
        -alias 'test-inc-root' -file ./data/private/root.ca.cert \
        -keystore "${JAVA_HOME}/lib/security/cacerts -storepass changeit

The location of the cacerts file may change a bit from version to version, but you can
always find it

.. code-block:: terminal

    $ find "${JAVA_HOME}" -type f -iname 'cacerts'

On mac you can probably get away with the following

.. code-block:: terminal

    $ find /Library/Java/JavaVirtualMachines -type f -iname cacerts | while read store; do
    >     keytool -import \
    >         -alias 'test-inc-root' -file ./data/private/root.ca.cert \
    >         -keystore "${store} -storepass changeit
    > done

.. warning::

    If you have installed gradle or ant with homebrew you will have a jdk installed by
    homebrew as well. When running ``gradle`` or ``ant`` the howbrew installed jdk will be
    used. This jer is **not** located in ``/Library/Java/JavaVirtualMachines``. You will
    thus need to add your certificate there if you do not have your java home set.

    This jdk is probably located under ``/usr/local/Cellor/openjdk/``.


