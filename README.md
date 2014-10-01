
# Steps to setup a vagrant box with MusicBrainz data in postgresql server

- Clone this project
- Create Ubuntu box with postgres by applying steps from 
[my fork of LaKIM's ruby box](https://github.com/gfauredumont/ruby-chef-box)

- Load DB dump from 'full_export' file
http://musicbrainz.org/doc/MusicBrainz_Database/Download
=>  http://ftp.musicbrainz.org/pub/musicbrainz/data/fullexport/
(copy into project dir)




- Create musicbrainz db in postgres by using 'CreateTables.sql'
Schema used to setup this bow come from MusicBrainz GitHub repo,
file (included in this repo as 'mb/schema.sql') was last updated on 2014 Jul 31st:

[MusicBrainz GH CreateTables.sql file path](https://github.com/metabrainz/musicbrainz-server/blob/master/admin/sql/CreateTables.sql)



```shell
$ mv \vagrant\CreateTables.sql \vagrant\mb_schema

# It's probably a good idea to upgrade your bow before seting everything up:
$ sudo apt-get upgrade

$ sudo apt-get install postgresql-contrib
$ sudo /etc/init.d/postgresql restart
```

The next steps are regarding postgres' setup so let's go to the mb directory:
```shell
cd /vagrant/mb
```


We'll now start connecting again and again in postgres, but for more and more precise tasks:
(password for postgres user is 'postgres')
```shell
$ psql --username=postgres --password --host=localhost
```

For now, just create the MB database:
```plsql
CREATE DATABASE musicbrainz ENCODING 'UTF8';
\q
```

Now we need to connect to this specific database....
```shell
$ psql --username=postgres --password --host=localhost -d musicbrainz
```

... in order to install the 'cube' extension in it:
```plsql
CREATE EXTENSION cube;
\q
```

So now we can run the Schema creation script in order to receive the data:
```shell
$ psql --username=postgres --password --host=localhost -d musicbrainz -f CreateTables.sql
```

At this point, we have created the MusicBrainz database and all its tables
We don't need the triggers and stuff as we don't intend on using the database, just reading it !


- Copy the MusicBrainz database dump into the /vagrant/mb directory
(should be faster to do this on the host side...)

- Extract the tarball into files
```shell
$ tar -xvf mbdumb.tar.bz2
```


- Load the MusicBrainz dump from files into postgres
(WARNING: this script loads all data files from '/vagrant/mb/mbdump'; if you've moved the file, please modify this script accordingly)
```shell
$ psql --username=postgres --password --host=localhost -d musicbrainz -f mb_load.sql
```


MODIFY postgresql to allow remote connection
This is ugly, but it works ... and we're in a VM... and we have firewall and stuff... :/

[REF](https://coderwall.com/p/cr2a1a)


```shell
$ sudo vi /etc/postgresql/9.1/main/pg_hba.conf
```

- ADD this line into the file 
```
host all all  0.0.0.0/0 md5
```

```shell
$ sudo vi /etc/postgresql/9.1/main/postgresql.conf
```
- REPLACE listen_addresses to listen on everything:
```
listen_addresses = '*'
```

- RESTART postgresql server to load modifications:
```shell
$ sudo service postgresql restart
```

Don't forget to update your host's firewall in case it's a bit restrictive ;)
