galera-secure-rsync
===================

Drop-in SSL-secured rsync SST script for Percona Cluster.

Why do I need / want this?
--------------------------

[Percona Cluster](http://www.percona.com/software/percona-xtradb-cluster) is awesome.  [Galera](http://codership.com/products/galera_replication) is awesome.  It all works great and is easy to setup, but there is no way to secure all the traffic between the nodes out of the box.  Enter galera-secure-rsync.

How does it work?
-----------------

Galera replication communication itself can be secured with SSL, but the SST traffic required to bootstrap a new node has no secure options out of the box. galera-secure-sync operates almost exactly like wsrep_sst_rsync except that it secures the actual communications with SSL using [socat](http://www.dest-unreach.org/socat/).  You generate a set of client/server credentials, pass them to every node, then change your my.cnf to use the new SST method and pass the directory where the credentials live as the auth parameter.  Voila, secured SST traffic to match your secured Galera replication.

How to set it up
----------------

_These instructions assume default script locations when following Percona's Percona Cluster set-up guide_

First, let's make sure we have openssl and socat.

    yum install socat openssl

Now grab a copy of the secure rsync SST script and move it into position.

    git clone git://github.com/tobz/galera-secure-rsync.git
    cd galera-secure-rsync
    cp wsrep_sst_secure_rsync /usr/bin
    
Now you'll need to generate the client/server credentials.

    openssl genrsa -out server.key 2048
    openssl req -new -key server.key -x509 -days 365000 -out server.crt -batch
    cat server.key server.crt >server.pem
    openssl genrsa -out client.key 2048
    openssl req -new -key client.key -x509 -days 365000 -out client.crt -batch
    cat client.key client.crt >client.pem
    chmod 400 client.* server.*
    chown mysql:mysql client* server.*
    
Move those credentials to a folder of your choosing and copy them to every node in the cluster.

    mkdir -p /etc/percona/ssl
    mv client.* /etc/percona/ssl
    mv server.* /etc/percona/ssl
    
Now, update your my.cnf.  Add/edit the follow to match the values below:

    wsrep_sst_method=secure_rsync
    wsrep_sst_auth=/etc/percona/ssl
    
Start up your nodes according to the Percona Cluster guide (first node to create the cluster, second node pointed at the first to join in) and your second node should connect securely over an SSL tunnel to complete the SST.  Voila! :)
    
    
    
