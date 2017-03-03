# solr-kerberos-docker
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
    # ant server
    # bin/solr -c -force

Verify that kerberos is working:

    # curl http://solr1:8983/solr/

(should throw 403 error)

    # kinit client@EXAMPLE.COM
    (password: luc1d)
    # curl --negotiate -u : http://solr1:8983/solr/
    
(should show the HTML of the admin UI)

Opening the Solr Admin UI from the browser of the host machine:

    TODO


Setting up Solr (say, on host machine) to use this KDC
------------------------------------------------------
If you are comfortable working inside the Solr development docker container that we created in the previous step, you don't need to go through all this.

    # vi /etc/krb5.conf

Add the following section:

    [realms]
     EXAMPLE.COM = {
      kdc = 172.17.0.2
      admin_server = 172.17.0.2
     }

Also, add the following line in the [libdefaults] section (create this section if not already there):

    default_realm = EXAMPLE.COM

Now, try to do kinit:

    $ kinit erik@EXAMPLE.COM

Setting up Solr dev environment:

    TODO
    
Note
----
1. Passwords are luc1d for principals: ishan, client
2. The keytabs file can be used for principals: HTTP/solr1, HTTP/solr2, zookeeper/zk1
3. Root password (for ssh) is: luc1d

