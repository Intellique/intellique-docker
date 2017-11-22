# Intellique Archival and Digital Vault Docker Distribution

This is the docker-compose file for *Intellique Open Digital Vault*.
It allows you to easily deploy and test our archival system using Docker.

### About Intellique ODV
To learn more about data archiving and Intellique ODV, [check our website](http://intellique.org).

The full appplication uses 6 docker images:

 * [Intellique NextCloud](https://hub.docker.com/r/intellique/nextcloud/)
   * this is combining NextCloud with an Intellique API client to interact with Intellique ODV.

 * [Intellique database](https://hub.docker.com/r/intellique/database/)
   * ths is the postgresql database with some minimal data filled in.

 * [Intellique api](https://hub.docker.com/r/intellique/api/)
   * this container provides the Intellique API documented [here](https://github.com/Intellique/api)

 * [Intellique daemon](https://hub.docker.com/r/intellique/daemon/)
   * the daemon is the program actually moving data around. Its source code is available [here](https://github.com/Intellique/daemon) 

 * [Intellique Web UI](https://hub.docker.com/r/intellique/webui/)
   * this is a simple web interface that consumes the API. Its source code is available [here](https://github.com/Intellique/webui) 

 * [HA Proxy](https://hub.docker.com/r/intellique/haproxy/)
   * this one manage http routing between NextCloud, the Web UI and the API.

## Architecture
   
Here is how this is all architectured.

![schema of containers interactions](https://wazoox.github.io/DOCK001.png)

## Deploy

1. Install [docker](https://docs.docker.com/engine/installation/) and [docker-compose](https://docs.docker.com/compose/install/). Minimal required version are Docker v1.10.0 and docker-compose v1.6.0. We tested with Docker 17.05 and docker-compose v1.8.0.
2. get the [docker-compose.yml](https://raw.githubusercontent.com/Intellique/intellique-docker/master/docker-compose.yml) file
3. run docker-compose:

    `docker-compose up -d`

## Using the application

The system is pre-configured with one user account, "storiq", using the password "spider77".

Go to `http://<docker host IP>/nextcloud` (or install and configure a nextcloud client) and connect as storiq.
Drop files into the nextcloud volume.

### Archiving

When you're ready to archive, move files into the *to_archive* folder; then create or upload in *to_archive* a file named *archive.txt* containing just one line with the name you want for your archive.
Wait for at most 15 minutes; the files will disappear from *to_archive*. They've been archived!

### Restoring

To restore files, go to `http://<docker host IP>/intellique` and connect as storiq.
The left pane allows you to switch between archives (an archive may contains many files), files, media (physical storage locations, may contain many archives), User account, and for authorized users, Administration.

Browse the archives or files, or search for archives or files. Click on the *+* sign to display more information. To restore an archive or file, click on the "Restore this archive" or "Restore this file" button.

Restored data appears in the Nextcloud volume in the *restored* folder. 

### Managing users

**Don't create users from the NextCloud interface**. Create users and change their passwords from the *Intellique* Web UI instead; this will also automatically create the corresponding user account in NextCloud.

Users created from the same *master account* have their own NextCloud space; but they all share the same archival pool. Therefore they can restore archives made by others.

## Using the application through the API

See the [API documentation](https://github.com/Intellique/api). It is recommended that you create a new API key for each application that will use it, so that proper information is logged.
To create an API key, connect to the docker image running the daemon:

    docker exec -i -t daemon /bin/bash
	
Then use the *storiqonectl* command this way:

    storiqonectl api --create <Your App Name>
	
Note carefully the API key returned. You can then use this API key to send request, for instance:

    # authenticate with the API
	curl -v -k -X POST --data '{"login":"admin","password":"spider77","apikey":"<uuid>"'}' \
		-H 'content-type: application/json' \
		http://<docker host IP>/api/v1/auth/
		
	# get information from archive id 1
	curl -v -k -X GET -H 'content-type: application/json' -b 'PHPSESSID=<string>' \
		http://<docker host IP>/api/v1/archive/?id=1
