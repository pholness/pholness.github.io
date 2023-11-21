---
layout: default
title: PostgreSQL
parent: Databases
nav_order: 7
---
# Setting up and using PostgreSQL on Linux Mint Debian Edition

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Linux Mint DE

Start with a base install of Linux Mint Debian Edition. I chose LMDE 6 Faye. After the install completes, allow software update to add any new packages.

## Installing PostgreSQL

Use the Software Manager tool in LMDE to install the Postgresql package. This sets everything up including the require scripts to stop and start the database cluster. 

## Install pgAdmin4 

Go to the pgAdmin4 download page for Linux APT downloads.

https://www.pgadmin.org/download/pgadmin-4-apt/

Follow the instructions, but when it comes to adding the repository, use the name for the Debian version LMDE is based on. In this case it was bookworm:

```
# Install the public key for the repository (if not done previously):
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg

# Create the repository configuration file:
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/bookworm pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'

#
# Install pgAdmin
#

# Install for desktop mode only:
sudo apt install pgadmin4-desktop
```

## Setting up users

By default you need to log on to the database as the postgres Unix user. For my testing I wanted to use a new user "peter", unrelated to any of the Unix accounts. I also wanted to create a new database "sandpit", with a new schema "test" to hold objects. The new user should be able to log onto this new database, and their default schema should be set to the new schema. This was achieved as follows.

As the postgres Unix user

```
psql 
postgres=# create database sandpit;
CREATE DATABASE
postgres=# create user peter login password 'topsecretpassword';
CREATE ROLE
postgres=# \c sandpit
You are now connected to database "sandpit" as user "postgres".
sandpit=# create schema test authorization peter;
CREATE SCHEMA
sandpit=# alter user peter set search_path to test;
ALTER ROLE
```

Now, as any unix user, connect to the database as the "peter" role. Note that the content of the /etc/postgresql/15/main/pg_hba.conf by default means that local connections automatically attempt to log into the database using the unix name as the database role. If you try this with psql you get an error.

As another Unix user:

```
$ psql sandpit peter
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "peter"
```

The pg_hba.conf file by default looks like this:

```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
```

Note the local all line uses peer as the authentication method. What we need to do in order to be able to provide our preferred database role / user to log with, is to connect via the network listener.

```
 psql sandpit peter -h localhost
Password for user peter: 
psql (15.5 (Debian 15.5-0+deb12u1))
Type "help" for help.
```

Once connected like this, the peter user can create objects in the test schema of the sandpit database.
