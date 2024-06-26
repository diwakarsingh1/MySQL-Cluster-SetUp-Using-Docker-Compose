# MySQL-Cluster-SetUp-Using-Docker-Compose

![](/images/MySQL-Docker.png)

__In this, I will take you through setting up a three node mysql innodb cluster using only docker containers which can be implemented anywhere. Mysql is one of the most prominent and popular database solutions adopted by corporate and companies for their applications or business needs. A cluster is a collection of resources or in this case servers which provides scalability and high availability. Mysql clusters can serve millions of users handle high volumes of data load, provides real time response and agility.__
<h2>Architecture of MYSQL Cluster.</h2>

![](/images/cluster.png)


- __To provision the mysql cluster we first require three standalone mysql servers which we will use docker in our case.__

- __The cluster has three nodes or containers in this case. A single primary node which has read-write privileges. Two secondaries in which they have read-only privileges.__

__Lets Configure setup.sql file.__
- To setup a cluster we create an user clusteradmin which has all privileges. This user is created on all nodes when the setup is initiated.

      CREATE USER 'clusteradmin'@'%' IDENTIFIED BY 'cladmin';
      GRANT ALL privileges ON *.* TO 'clusteradmin'@'%' with grant option;
      reset master;

- mysql-server:8.0 is used and a sql data file is used to perform some initial setup. The file is copied to docker-entrypoint-initdb.d which will ensure to execute the content of the file.

__Lets Write Dockerfile.__
- To provision the mysql cluster we first require three standalone mysql servers which we will use docker in our case. To being we define a dockerfile which will be used as our base image

      FROM mysql/mysql-server:8.0
      COPY ./setup.sql /docker-entrypoint-initdb.d
      EXPOSE 3306

__Finally a docker-compose file is used to create the docker containers.__

- docker-compose.yml

      version: "3.3"
      services:
        mysql-dev1:
            build: .
            command: --default-authentication-plugin=mysql_native_password
            environment:
              MYSQL_ROOT_PASSWORD: password
            volumes:
            - ./db-data1:/var/lib/mysql
            ports:
            - "3306:3306"
  
        mysql-dev2:
            build: .
            command: --default-authentication-plugin=mysql_native_password
            environment:
              MYSQL_ROOT_PASSWORD: password
            ports:
            - "3307:3306"
            volumes:
            - ./db-data2:/var/lib/mysql
  
        mysql-dev3:
            build: .
            command: --default-authentication-plugin=mysql_native_password
            environment:
              MYSQL_ROOT_PASSWORD: password
            ports:
            - "3308:3306"
            volumes:
            - ./db-data3:/var/lib/mysql

      volumes:
        db-data1:
          driver: local
        db-data2:
          driver: local
        db-data3:
          driver: local
    
- Three docker containers have been setup which have the root password as password with three different port number mappings to the host system and default authentication plugin as password.

- Ensure to set a volume for the containers as this is where mysql stores its metadata.

- __Start the three servers by running the docker compose command.__

      docker-compose up -d


![](/images/docker-containers.webp)

- This starts up three docker containers which will be our mysql servers also parallely creating three images for the containers.

- Exec into one of the nodes as root user and view the user,host present to ensure clusteradmin user is present.

- Exec into one of the nodes as root user.

      docker exec -it {container name} /bin/bash   // exec into container

- LogIn into MySQL.

      mysql -uroot -p'password'  // Login as root user

- Check clusteradmin user present or not.

      SELECT user,host FROM mysql.user;     // list all users and host

![](/images/clusteradmin.webp)

- Login as clusteradmin user to ensure user has been setup with correct credentials.

      mysql -uclusteradmin -p'cladmin'; // login as cluster admin

![](/images/cladmin.webp)

- Considering one of the nodes as primary use mysqlsh to initiate the cluster creation process.
- Login to mysql shell using clusteradmin user.

      mysqlsh -uclusteradmin -p'cladmin'

![](/images/cluster-login.webp)

- The first step is to check the node configuration if it follows the required requirements.
- On same shell run the below command replacing container names with the names of the docker containers since each container has connectivity to each other.

      dba.checkInstanceConfiguration("clusteradmin@{container name}:port")

![](/images/status-check.webp)

- From the single mysqlsh shell the other nodes can also be checked by replacing the container names with corresponding container names.
- Each container is then configured so that they can act as part of an innodb cluster.
- On same shell run the below command replacing container names with the names of the docker containers.

      dba.configureInstance("clusteradmin@{container name}:port")

![](/images/setting-status.webp)

- This will setup the needed environment variables and settings so that each container can act as an innodb node.
- The configureInstance command must be run three times for each container with the respective container name , this can be run from the same mysql shell .
- On running the checkInstanceConfiguration we notice that it results in a success now.

![](/images/status-ok.webp)

__Creating the cluster.__

- On mysql shell initiate cluster creation.

      var cluster = dba.createCluster("{cluster name}")

![](/images/cluster-create.webp)

- Check cluster status.

      cluster.status()

![](/images/cluster-status.webp)

- We have a single node cluster in which we need to add the other two containers.
- Add a different instance/container using the addInstance command.

      cluster.addInstance("clusteradmin@{container name}:{port}")

![](/images/add-container.webp)

- This command has to be used to add the other two containers to the primary node.
- Running cluster.status() after each step will allow us to understand the cluster status.

![](/images/all-node.webp)

- We have a three node mysql with docker setup with one primary and two secondaries. Primary is used for all write operations which is replicated to the secondaries.

__Features of the mysql cluster.__

__1. Group Replication__
- As per the architecture the primary acts as the node which is responsible for writes in the database which is replicated to the secondaries.

- Login to mysql primary node and enter data which is replicated to other secondaries.

__2. Automatic fail over__
- If for any case one the secondary nodes fails the cluster is still up and running , however if a primary node fails a new primary is elected and operations continue.
- We go ahead and stop and start the primary node container.
- In such situation the node goes to status missing and another node is selected as primary.

![](/images/missing.webp)

- Once node comes back up it rejoins the cluster.

![](/images/rejoin.webp)

__Errors or debugging.__

- In some cases it may be required to perform a cluster rescan on node addition or after a fail over.
- Run the command.

      var cluster = dba.getCluster()  // if cluster var is not set
      cluster.rescan()

<h1> Voila!💥 Your MySQL-Cluster-SetUp-Using-Docker-Compose is ready.</h1>