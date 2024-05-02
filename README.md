# MySQL-Cluster-SetUp-Using-Docker-Compose

![](/images/MySQL-Docker.png)

__In this, I will take you through setting up a three node mysql innodb cluster using only docker containers which can be implemented anywhere. Mysql is one of the most prominent and popular database solutions adopted by corporate and companies for their applications or business needs. A cluster is a collection of resources or in this case servers which provides scalability and high availability. Mysql clusters can serve millions of users handle high volumes of data load, provides real time response and agility.__
<h2>Architecture of MYSQL Cluster.</h2>

![](/images/cluster.png)


- __To provision the mysql cluster we first require three standalone mysql servers which we will use docker in our case.__

- __The cluster has three nodes or containers in this case. A single primary node which has read-write privileges. Two secondaries in which they have read-only privileges.__

__Lets Configure setup.sql file.__
To setup a cluster we create an user clusteradmin which has all privileges. This user is created on all nodes when the setup is initiated.

      CREATE USER 'clusteradmin'@'%' IDENTIFIED BY 'cladmin';
      GRANT ALL privileges ON *.* TO 'clusteradmin'@'%' with grant option;
      reset master;

mysql-server:8.0 is used and a sql data file is used to perform some initial setup. The file is copied to docker-entrypoint-initdb.d which will ensure to execute the content of the file.

__Lets Write Dockerfile.__
To provision the mysql cluster we first require three standalone mysql servers which we will use docker in our case. To being we define a dockerfile which will be used as our base image

      FROM mysql/mysql-server:8.0
      COPY ./setup.sql /docker-entrypoint-initdb.d
      EXPOSE 3306

- __Finally a docker-compose file is used to create the docker containers.__

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
            - "3305:3306"
  
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
    
