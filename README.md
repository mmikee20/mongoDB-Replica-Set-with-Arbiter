# mongoDB-Replica-Set-with-Arbiter
Build a 3-Node MongoDB Replica Set with Arbiter Locally
================================================================

This task introduces you to MongoDB replica sets, which provide high availability and data redundancy through automatic failover and data replication. You will learn how to use Docker Compose to orchestrate a MongoDB cluster with three data-bearing nodes and one arbiter node.

You will also gain experience in:

MongoDB replication and automatic elections

Understanding arbiter nodes (no data, vote-only)

Container networking and configuration in Docker Compose

Volume persistence and container initialization

Using the MongoDB shell (mongosh) to initiate and inspect replica sets

Objectives
Deploy a 3-node MongoDB replica set using Docker Compose, with one additional arbiter node. All nodes should be part of a properly initialized replica set with replication enabled and elections supported via the arbiter.

Requirements
Use official mongo:7 image.

Create 3 data-bearing nodes: mongo1, mongo2, mongo3

Create 1 arbiter node: arbiter

All nodes should:

Share the same replica set name (e.g., rs0)

Use persistent storage

Communicate over a custom Docker bridge network

Use mongosh to initiate the replica set with roles: 2 secondaries, 1 arbiter

Docker Compose File
=================================================
version: '3.8'

services:
  mongo1:
    image: mongo:7
    container_name: mongo1
    ports:
      - "27017:27017"
    volumes:
      - ./mongo1/data:/data/db
    networks:
      mongo-cluster:
        ipv4_address: 172.26.0.11
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]

  mongo2:
    image: mongo:7
    container_name: mongo2
    ports:
      - "27018:27017"
    volumes:
      - ./mongo2/data:/data/db
    networks:
      mongo-cluster:
        ipv4_address: 172.26.0.12
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]

  mongo3:
    image: mongo:7
    container_name: mongo3
    ports:
      - "27019:27017"
    volumes:
      - ./mongo3/data:/data/db
    networks:
      mongo-cluster:
        ipv4_address: 172.26.0.13
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]

  arbiter:
    image: mongo:7
    container_name: arbiter
    networks:
      mongo-cluster:
        ipv4_address: 172.26.0.14
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all", "--nojournal"]

networks:
  mongo-cluster:
    driver: bridge
    ipam:
      config:
        - subnet: 172.26.0.0/24
Replica Set Initialization
Once all containers are up, exec into mongo1:



docker exec -it mongo1 mongosh
Then run:

rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" },
    { _id: 3, host: "arbiter:27017", arbiterOnly: true }
  ]
})

Check status:
==================
rs.status()


Learning material
=========================
https://www.mongodb.com/docs/manual/replication/
https://www.mongodb.com/docs/manual/core/replica-set-arbiter/
https://www.mongodb.com/docs/mongodb-shell/



Usefull stuff
==============
MongoDB replica sets:

Primary
Handles all writes (insert, update, delete).
By default, most applications also read from the primary to ensure the freshest data.

Secondaries
Continuously replicate the data from the primary using the oplog (operation log).
They normally don’t accept writes (unless you configure “read preference” to allow reads from them, but then the data may be slightly behind = eventual consistency).
If the primary goes down, one secondary is elected automatically to become the new primary.

Arbiter:
Does not store data, but participates in voting during elections.
Useful if you want an odd number of voting members without the cost of another full data node.

👉 So, yes:

Clients read/write from the primary.
Secondaries keep redundant copies for fault tolerance and high availability.
In failover, a secondary steps up and becomes the new primary.


docker exec -it mongo1 bash
mongosh

or
docker exec -it mongo1 mongosh


rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" },
    { _id: 3, host: "arbiter:27017", arbiterOnly: true }
  ]
})

rs.status()


Commands:
------------------------
warning: db is a special variable provided by mongosh that always points to the current database you are using. So we have to always use the db in the commands.

rs.status().members.filter(m => m.stateStr === "PRIMARY")[0].name		(check who is the primary in order to connect)

show dbs	(list dbs)
db	(check what db you are using)
use name_db	(use the db)
show collections	(show tables)

db.users.insertOne({name: "Alice", age: 25})
db.users.insertMany([{name: "Bob"}, {name: "Charlie"}])

db.users.find()
db.users.find({age: {$gt: 20}})

db.users.updateOne({name: "Alice"}, {$set: {age: 26}})

db.users.deleteOne({name: "Charlie"})

db.users.find()	(see some data)

