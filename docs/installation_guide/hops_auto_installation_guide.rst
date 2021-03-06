.. _hops-installer:

*******************
Hops Installation
*******************

The Hops stack includes a number of services, also requires a number of third-party distributed services:

* Java 1.7+ (OpenJDK or Oracle JRE/JDK)
* NDB 7.5 (MySQL Cluster)
* J2EE7 web application server (default: Glassfish)
* ElasticSearch 1.7+
  
Due to the complexity of installing and configuring all Hops' services, we recommend installing Hops using the automated installer Karamel/Chef (http://www.karamel.io). We do not provide detailed documentation on the steps for installing and configuring all services in Hops. Instead, Chef cookbooks contain all the installation and configuration steps needed to install and configure Hops. The Chef cookbooks are available at: https://github.com/hopshadoop.


Cloud Platforms (AWS, GCE, OpenStack)
-------------------------------------

Hops can be installed on a cloud platform using Karamel/Chef.


Karamel/Chef
~~~~~~~~~~~~

#. Download and install Karamel (http://www.karamel.io).
#. Run Karamel.
#. Click on the "Load Cluster Definition" menu item in Karamel. You are now prompted to select a cluster definition YAML file. Go to the examples/stable directory, and select a cluster definition file for your target cloud platform for one of the following cluster types:

   #. Amazon Web Services EC2 (AWS)
   #. Google Compute Engine (GCE)
   #. OpenStack
   #. On-premises (bare metal)

For more information on how to configure cloud-based installations, go to help documentation at http://www.karamel.io.
For on-premises installations, we provide some additional installation details and tips later in this section.


**Choosing which services to run on which nodes**

You now need to decide which services you will install on which nodes. In Karamel, we design a set of *Node Groups*, where each *Node Group* defines a stack of services to be installed on a machine. Each machine will only have one *Node Group* set of services.
We now provide two recommended setup:

* a *single node cluster* that includes all services on a single node.
* a *tiny cluster* set of *heavy* stacks that includes a lot of services on each node.
* a *small cluster* set of *heavy* stacks that includes lots of services on each node.
* a *large cluster* set of *light* stacks that includes fewer services on each node.

**Single Node Setup**
You can run the entire HopsWorks application platform on a single node. You will have a NodeGroup with the following services on the single node:

#. HopsWorks, Elasticsearch, Zeppelin, MySQL Server, NDB Mgmt Server, HDFS NameNode, YARN ResourceManager, NDB Data Node(s), HDFS DataNode, YARN NodeManager


**Tiny Cluster Setup**

We recommend the following setup that includes the following NodeGroups, and requires at least 2 nodes to be deployed:

#. HopsWorks, Elasticsearch, Zeppelin, MySQL Server, NDB Mgmt Server, HDFS NameNode, YARN ResourceManager, NDB Data Node
#. HDFS DataNode, YARN NodeManager

This is really only a test setup, but you will have one node dedicated to YARN applications and file storage, while the other node handles the metadata layer services.


**Small Cluster Setup**

We recommend the following setup that includes four NodeGroups, and requires at least 4 nodes to be deployed:

#. HopsWorks, Elasticsearch, Zeppelin, MySQL Server, NDB Mgmt Server, 
#. HDFS NameNode, YARN ResourceManager, MySQL Server 
#. NDB Data Node
#. HDFS DataNode, YARN NodeManager    

A highly available small cluster would require at least two instances of the last three NodeGroups. HopsWorks can also be deployed on mulitple instances, but Elasticsearch needs to be specially configured if it is to be sharded across many insances.
  
**Large Cluster Setup**

We recommend the following setup that includes six NodeGroups, and requires at least 4 nodes to be deployed:

#. Elasticsearch
#. HopsWorks, Zeppelin, MySQL Server, NDB Mgmt Server
#. HDFS NameNode, MySQL Server
#. YARN ResourceManager, MySQL Server       
#. NDB Data Node
#. HDFS DataNode, YARN NodeManager    

A highly available large cluster would require at least two instances of every NodeGroup. HopsWorks can also be deployed on mulitple instances, while Elasticsearch needs to be specially configured if it is to be sharded across many insances. Otherwise, the other services can be easily scaled out by simply adding instances in Karamel. For improved performance, the metadata layer could be deployed on a better network (10 GbE at the time of writing), and the last NodeGroup (DataNode/NodeManager) instances could be deployed on cheaper network infrastructure (bonded 1 GbE  or 10 GbE, at the time of writing).


**HopsWorks Configuration in Karamel**

Karamel Chef recipes support a large number of parameters that can be set while installing Hops. These parameters include, but are not limited to,:

* usernames to install and run services as,
* usernames and passwords for services, and
* sizing and tuning configuration parameters for services (resources used, timeouts, etc).
    

Here are some of the most important security parameters to set when installing services:

* Superuser username and password for the MySQL Server(s)
  * Default: 'kthfs' and 'kthfs'
* Administration username and password for the Glassfish administration account(s)
  * Default: 'adminuser' and 'adminpw'      
* Administration username and password for HopsWorks
  * Default: 'admin@kth.se' and 'admin'            

Here are some of the most important sizing configuration parameters to set when installing services:

* DataMemory for NDB Data Nodes 
* YARN NodeManager amount of memory and number of CPUs
* Heap size and Direct Memory for the NameNode
* Heap size for Glassfish
* Heap size for Elasticsearch


On-Premises (baremetal) Installation
-------------------------------------

For on-premises (bare-metal) installations, you will need to prepare for installation by:

#. identifying a *master* host, from which you will run Karamel;

   #. the *master* must have a display for Karamel's user interface;
   #. the *master* must be able to ping (and connect using ssh) to all of the *target* hosts.
   
#. identifying a set of *target* hosts, on which the Hops software and 3rd party services will be installed.

   #. the *target* nodes should have http access to the open Internet to be able to download software during the installation process. (Cookbooks can be configured to download software from within the private network, but this requires a good bit of configuration work for Chef attributes, changing all download URLs).


The *master* must be able to connect using SSH to all the *target* nodes, on which the software will be installed. If you have not already copied the *master's* public key to the *.ssh/authorized_keys* file of all *target* hosts, you can do so by preparing the machines as follows:

#. Create an openssh public/private key pair on the *master* host for your user account. On Linux, you can use the ssh-keygen utility program to generate the keys, which will by default be stored in the ``$HOME/.ssh/id_rsa and $HOME/.ssh/id_rsa.pub`` files. If you decided to enter a password for the ssh keypair, you will need to enter it again in Karamel when you reach the ``ssh`` dialog, part of Karamel's ``Launch`` step. We recommend no password (passwordless) for the ssh keypair.
#. Create a user account ``USER`` on the all the *target* machines with full sudo privileges (root privileges) and the same password on all *target* machines. 
#. Copy the $HOME/.ssh/id_rsa.pub file on the *master* to the /tmp folder of all the *target* hosts. A good way to do this is to use ``pscp`` utility along with a file (``hosts.txt``) containing the line-separated hostnames (or IP addresss) for all the *target* machines. You may need to install the pssh utility programs (``pssh``), first.

.. code-block:: bash   

   $sudo apt-get install pssh
   or
   $yum install pssh
 
   $vim hosts.txt
      # Enter the row-separated IP addresses of all target nodes in hosts.txt
           128.112.152.122
           18.31.0.190
           128.232.103.201
           .....

   $pscp -h hosts.txt -P PASSWORD -i USER ~/.ssh/id_rsa.pub /tmp
   $pssh -h hosts.txt -i USER -P PASSWORD mkdir -p /home/USER/.ssh
   $pssh -h hosts.txt -i USER -P PASSWORD cat /tmp/id_rsa.pub 
          >> /home/USER/.ssh/authorized_keys
   
Update your Karamel cluster definition file to include the IP addresses of the *target* machines and the ``USER`` account name. After you have clicked on the ``launch`` menu item, you will come to a ``Ssh`` dialog. On the ``ssh`` dialog, you need to open the advanced section. Here, you will need to enter the password for the ``USER`` account on the *target* machines (``sudo password`` text input box). 
If your ssh keypair is password protected, you will also need to enter it again here in the ``keypair password`` text input box.


:Note: `Redhat/Centos is not yet supported by Karamel, but you can install Hops using Chef-solo by logging into each machine separately. The chef cookbooks are written to work for both the Debian/Ubuntu and Redhat/Centos platform families.`


Vagrant (Virtualbox)
-----------------------

You can install HopsWorks and Hops on your laptop/desktop  with Vagrant. You will need to have the following software packages installed:

* chef-dk, version >0.5+ (but not >0.8+)
* git
* vagrant
* vagrant omnibus plugin    
* virtualbox

  You can now run vagrant, using:
  

.. code-block:: bash     

    $ sudo apt-get install virtualbox vagrant		
    $ vagrant plugin install vagrant-omnibus
    $ git clone https://github.com/hopshadoop/hopsworks-chef.git
    $ cd hopsworks-chef
    $ berks vendor cookbooks
    $ vagrant up

Default Setup
*****************
    
You can then access Hopsworks from your browser at http://127.0.0.1:8080/hopsworks. The default credentials are:

::

  username: admin@kth.se
  password: admin

You can access the Hopsworks administration application from your browser at http://127.0.0.1:8080/hopsworks/index.xhtml. The default credentials are:

::

  username: admin@kth.se
  password: admin
  
  
The Glassfish web application server is also available from your browser at http://127.0.0.1:4848. The default credentials are:

::

  username: adminuser
  password: adminpw


The MySQL Server is also available from the command-line, if you ssh into the vagrant host (``vagrant ssh``). The default credentials are:

::

  username: kthfs
  password: kthfs

It goes without saying, but for production deployments, we recommend changing all of these default credentials. The credentials can all be changed in Karamel during the installation phase.  
  
Windows
-------

You can also install HopsWorks on vagrant and Windows. You will need to follow the vagrant instructions as above (installing the same software packages) aswell as installing:

* Powershell

After cloning the github repo, from the powershell, you can run:

.. code-block:: bash     

    $ cd hopsworks-chef
    $ berks vendor cookbooks
    $ vagrant up
  
Apple OSX/Mac
-------------------


First, install protobuf and cmake:

.. code-block:: bash

    $ brew install protobuf250
    $ brew install cmake

Note: you need protobuf version 2.5 (not 2.6). You can check if it's already installed with

.. code-block:: bash

    $ protoc --version
    libprotoc 2.5.0

Hops runs on JDK 1.7, so you should have that installed (download it `here`_). However, due to some inconsistencies between Java version you'll have to create a symlink like this:

.. code-block:: bash

    cd /Library/Java/JavaVirtualMachines/jdk1.7.0_17.jdk/Contents/Home/
    sudo mkdir Classes && cd Classes
    sudo ln -s ../jre/lib/rt.jar classes.jar

.. _here: http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html

You should now be able to clone the repositories and build the code.

Hops Chef Cookbooks
--------------------

Hops' automated installation is orchestrated by Karamel and the installation/configuration logic is written as ruby programs in Chef. Chef supports the modularization of related programs in a unit of software, called a Chef *cookbook*. A Chef cookbook can be seen as a collection of programs, where each program contains instructions for how to install and configure software services. A cookbook may consist one or more programs that are known as *recipes*. These Chef recipes are executed by either a Chef client (that can talk to a Chef server) or chef-solo, a standalone program that has no dependencies on a Chef Server. Karamel uses chef-solo to execute Chef recipes on nodes. The benefit of this approach is that it is agentless. That is, Karamel only needs ssh to be installed on the target machine to be able to install and setup Hops. Karamel also provides dependency injection for Chef recipes, supplying the parameters (*Chef attributes*) used to execute recipes. Some stages/recipes return results (such as the IP address of the NameNode) that are used in subsequent recipes (for example, to generate configuration files containing the IP address of the NameNode, such as core-site.xml).

The following is a brief description of the Chef cookbooks that we have developed to support the installation of Hops. The recipes have the naming convention: <cookbook>/<recipe>. You can determine the URL for each cookbook by prefixing the name with http://github.com/. All of the recipes have been *karamelized*, that is a Karamelfile containing orchestration rules has been added to all cookbooks.


* hopshadoop/apache-hadoop-chef

   * This cookbook contains recipes for installing the Apache Hadoop services: HDFS NameNode (hadoop::nn), HDFS DataNode (hadoop::dn), YARN ResourceManager (hadoop::rm), YARN NodeManager (hadoop::nm), Hadoop Job HistoryServer for MapReduce (hadoop::jhs), Hadoop ProxyServer (hadoop::ps).
  
* hopshadoop/hops-hadoop-chef

   * This cookbook contains is a wrapper cookbook for the Apache Hadoop cookbook. It install Hops, but makes use of the Apache Hadoop Chef cookbook to install and configure software. The recipes it provides are: HopsFS NameNode (hops::nn), HopsFS DataNode (hops::dn), HopsYARN ResourceManager (hops::rm), HopsYARN NodeManager (hops::nm), Hadoop Job HistoryServer for MapReduce (hops::jhs), Hadoop ProxyServer (hops::ps).

* hopshadoop/elasticsearch-chef

   * This cookbook is a wrapper cookbook for the official Elasticsearch Chef cookbook, but it has been extended with Karamel orchestration rules. 
  
* hopshadoop/ndb-chef

   * This cookbook contains recipes for installing MySQL Cluster services: NDB Management Server (ndb::mgmd), NDB Data Node (ndb::ndbd), MySQL Server (ndb::mysqld), Memcached for MySQL Cluster (ndb::memcached).
  
* hopshadoop/zeppelin-chef

   * This cookbook contains a default recipe for installing Apache Zeppelin.

* hopshadoop/hopsworks-chef

   * This cookbook contains a default recipe for installing HopsWorks.
  
* hopshadoop/spark-chef

   * This cookbook contains recipes for installing the Apache Spark Master, Worker, and a YARN client.

* hopshadoop/flink-chef

   * This cookbook contains recipes for installing the Apache Flink jobmanager, taskmanager, and a YARN client.

* hopshadoop/dr-elephant-chef

   * This cookbook contains recipes for installing Dr Elephant.

* hopshadoop/livy-chef

   * This cookbook contains recipes for installing Livy REST Server for Spark.

* hopshadoop/epipe-chef

   * This cookbook contains recipes for installing ePipe, exporting HopsFS' namespace to Elasticsearch for free-text search of the HDFS namespace.

* hopshadoop/dela-chef

   * This cookbook contains recipes for installing dela, the peer-to-peer tool for sharing datasets in Hopsworks.

* hopshadoop/hopsmonitor-chef

   * This cookbook contains recipes for installing tools for logging streaming applications - Kibana and Logstash.

* hopshadoop/tensorflow-chef

   * This cookbook contains recipes for installing tensorflow to work with Hopsworks and Zeppelin.
  
  

BiobankCloud Chef Cookbooks
--------------------------

* biobankcloud/charon-chef
  This cookbook contains a default recipe for installing Charon.

* biobankcloud/hiway-chef
  This cookbook contains recipes for installing HiWAY, Cuneiform, the BiobankCloud workflows, and some example workflows.

* hopshadoop/shibboleth-chef
  This cookbook contains a default recipe for installing Shibboleth. More documentation is available at: http://www.hops.io/?q=shibboleth

