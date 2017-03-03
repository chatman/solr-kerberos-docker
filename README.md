# solr-kerberos-docker
Download docker image and keytab files
--------------------------------------
    wget http://185.14.187.116/keytabs

    wget http://185.14.187.116/kdc-docker-image.tar.xz


Loading the image and creating a container
------------------------------------------

    $ docker load < kdc-docker-image.tar.xz
    $ docker run -it --name kdc -h kdc kdc

Inside the container created:

    # service krb5-kdc start
    # service krb5-admin-server start
    # service ssh start

To check the ip address of this container, do this inside the container:

    # ifconfig

Assume the ip address is 172.17.0.2

Using a Solr development docker image along with the KDC image
--------------------------------------------------------------

TODO

Setting up Solr (say, on host machine) to use this KDC
------------------------------------------------------

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

Note
----
1. Passwords are luc1d for principals: ishan, client
2. The keytabs file can be used for principals: HTTP/solr1, HTTP/solr2, zookeeper/zk1
3. Root password (for ssh) is: luc1d

