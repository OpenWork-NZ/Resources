---
layout: page
title: Installing GeoNetwork
description: Instructions for how to install GeoNetwork 3.4.4 on a Debian server.
date: 2019-04-24 15:15:18 1200
---

# Installing GeoNetwork on a server

1. Install the dependencies:

    `# apt install openjdk-8-jre-headless tomcat8 postgis`

Using `-headless` on `openjdk-8-jre` is important because that prevents APT from
installing a whole desktop environment as a dependency.

2. Switch to the postgres user to configure your database:

    `# su postgres`

3. Create your database:

    `$ createdb -O postgres gn -E utf-8`

4. Enable PostGIS and PostGIS topology on your database:

    `$ psql -U postgres -d gn -c 'CREATE EXTENSION postgis'`

    `$ psql -U postgres -d gn -c 'CREATE EXTENSION postgis_topology'`

5. Set a password on the postgres account:

    `$ psql`

    `=# \password postgres`

6. Download geonetwork.war version 3.44, outside the postgres account.

    `$ exit`

    `$ wget https://versaweb.dl.sourceforge.net/project/geonetwork/GeoNetwork_opensource/v3.4.4/geonetwork.war`

Make sure this is placed in /var/lib/tomcat8/webapps/, so Tomcat will now where to look for it.

From now on I will refer to /var/lib/tomcat8/ as "$TOMCAT".

7. Edit $TOMCAT/webapps/geonetwork/WEB-INF/config-node/srv.xml, to tell it to use PostGIS.
    a. Comment out `<import resource="../config-db/h2.xml"/>`
    b. Uncomment `<import resource="../config-db/postgres-postgis.xml"/>`.

8. Edit $TOMCAT/webapps/geonetwork/WEB-INF/config-db/jdbc.properties, to configure the correct user account.

Change the values for `jdbc.username` and `jdbc.password` to match what you configured earlier.

9. Download to extension to server
10. unarchive it with `unzip`. You may need to install `unzip`, because `gunzip` won't work.
11. Copy the directory to $TOMCAT/webapps/geonetwork/WEB-INF/data/config/schema_plugins/ , and change the owner to `tomcat8`
12. Restart tomcat

    `# systemctl restart tomcat8.service`

You may need to place some .jar files in $TOMCAT/webapps/geonetwork/WEB-INF/lib/ .

13. Now install nginx in order to use it to optimize and secure public requests to the Tomcat server.

    `# sudo apt install nginx`

If you want you may test that this works by pointing your browser to this server, without specifying a port. You should get message telling you that nginx is running successfully.

14. Edit `/etc/nginx/sites-enabled/default` to tell it to pass requests on to Tomcat.

Replace the `location /` block with:

    location / {
        proxy_pass http://127.0.0.1:8080/;
    }

15. Restart nginx:

    `# systemctl restart nginx.service`
