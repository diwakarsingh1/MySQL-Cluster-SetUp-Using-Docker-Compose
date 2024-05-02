# MySQL-Cluster-SetUp-Using-Docker-Compose

![](/images/MySQL-Docker.png)

__In this, I will take you through setting up a three node mysql innodb cluster using only docker containers which can be implemented anywhere. Mysql is one of the most prominent and popular database solutions adopted by corporate and companies for their applications or business needs. A cluster is a collection of resources or in this case servers which provides scalability and high availability. Mysql clusters can serve millions of users handle high volumes of data load, provides real time response and agility.__
<h2>Architecture of MYSQL Cluster.</h2>

![](/images/cluster.png)


- __To provision the mysql cluster we first require three standalone mysql servers which we will use docker in our case.__
- __The cluster has three nodes or containers in this case. A single primary node which has read-write privileges. Two secondaries in which they have read-only privileges.__




__Lets Configure setup.sql file.__



