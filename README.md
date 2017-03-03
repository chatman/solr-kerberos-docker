# solr-kerberos-docker
Start Docker
------------------------
This works on Docker 1.12. Not sure if more recent versions work as is.

    $ sudo service docker start

Using a KDC docker image
------------------------
Download KDC docker image and keytab files:

    wget http://185.14.187.116/keytabs

    wget http://185.14.187.116/kdc-docker-image.tar.xz


Loading the image and creating a container:

    $ docker load < kdc-docker-image.tar.xz
    $ docker run -it --name kdc -h kdc kdc

Inside the container created:

    # service krb5-kdc start
    # service krb5-admin-server start
    # service ssh start

To check the ip address of this container, do this inside the container:

    # ifconfig

Assume the ip address is 172.17.0.2. Assume this terminal window is called window1.

Note:

    * Passwords are luc1d for principals: ishan, client
    * The keytabs file can be used for principals: HTTP/solr1, HTTP/solr2, zookeeper/zk1
    * Root password (for ssh) is: luc1d

Using a Solr development docker image along with the KDC image
--------------------------------------------------------------

Download docker image (do this in a different terminal window, lets call it window2):

    $ wget http://185.14.187.116/solr-dev-docker-image.tar.xz

Load the image and create a container:

    docker load < solr-dev-docker-image.tar.xz
    docker run -it --name solr1 -h solr1 solr-dev

Connect the containers with a common network. Do the following in a different terminal window (lets call it window3):

    docker network create solr-kerberos
    docker network connect solr-kerberos kdc
    docker network connect solr-kerberos solr1

Back inside the solr1 container (terminal window2), compile and run Solr:

    # cd lucene-solr
    # git pull
    # cd solr
    # ant server
    # bin/solr -c -force

Verify that kerberos is working:

    # curl http://solr1:8983/solr/

(should throw 403 error)

    # kinit client@EXAMPLE.COM
    (password: luc1d)
    # curl --negotiate -u : http://solr1:8983/solr/
    
(should show the HTML of the admin UI)

Setting up browser on host machine to access Solr Admin UI
----------------------------------------------------------
Install kerberos client:

    OS X: Pre-installed? (verify that /etc/krb5.conf file exists)
    Fedora: sudo yum install krb5-workstation, 
    Ubuntu: sudo apt-get install krb5-user)

Now add the KDC details:

    $ sudo vi /etc/krb5.conf

Add the following section (check the KDC's IP address from the first step, assuming 172.17.0.2):

    [realms]
     EXAMPLE.COM = {
      kdc = 172.17.0.2
      admin_server = 172.17.0.2
     }

Also, add the following line in the [libdefaults] section (create this section if not already there):

    default_realm = EXAMPLE.COM

Now, try to do kinit:

    $ kinit client@EXAMPLE.COM
    password: luc1d

Now, we have to add a hostname entry for solr1. First, find out the IP address of the solr1 container (inside window2):

    # ip addr
    
(assume the IP address is 172.17.0.3)

Now, in the host machine, open a new terminal window (say, window4) and add the host name entry:

    $ sudo vi /etc/hosts
    Add the following line:
    solr1  172.17.0.3
    
Open Firefox, and goto "about:config". Configure "network.negotiate-auth.delegation-uris" and "network.negotiate-auth.trusted-uris" as follows:

![Firefox Kerberos configuration](http://185.14.187.116/firefox-kerberos.png)


Now, access http://solr1:8983/solr and the Solr Admin UI should show up.
