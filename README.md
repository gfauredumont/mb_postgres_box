
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
file (included in this repo as 'mb/schema.sql') was last updated on 2014 Jul 31st
[MusicBrainz GH CreateTables.sql file path](https://github.com/metabrainz/musicbrainz-server/blob/master/admin/sql/CreateTables.sql)



```shell
$ mv \vagrant\CreateTables.sql \vagrant\mb_schema

# It's probably a good idea to upgrade your bow before seting everything up:
$ sudo apt-get upgrade

$ sudo apt-get install postgresql-contrib
$ sudo /etc/init.d/postgresql restart
```

sudo su - deploy
cd /vagrant

(password for postgres user is 'postgres')

psql --username=postgres --password --host=localhost
  CREATE DATABASE musicbrainz ENCODING 'UTF8'
  \q
psql --username=postgres --password --host=localhost -d musicbrainz
  CREATE EXTENSION cube;
  \q

psql --username=postgres --password --host=localhost -d musicbrainz -f mb_schema

=> CREATES THE MUSIC_BRAINZ DB !!


mkdir mb
cd mb
tar -xvf mbdumb.tar.bz2


```
psql --username=postgres --password --host=localhost -d musicbrainz -f mb_load.sql
```


MODIFY postgresql to allow remote connection
This is ugly, but it works ... and we're in a VM... and we have firewall and stuff... :/
https://coderwall.com/p/cr2a1a


sudo vi /etc/postgresql/9.1/main/pg_hba.conf
```
# /etc/postgresql/9.1/main/pg_hba.conf    ADD
host all all  0.0.0.0/0 md5
```

sudo vi /etc/postgresql/9.1/main/postgresql.conf
```
# /etc/postgresql/9.1/main/postgresql.conf
listen_addresses = '*'
```

sudo service postgresql restart
