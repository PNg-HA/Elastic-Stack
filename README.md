[![Elastic Stack version](https://img.shields.io/badge/Elastic%20Stack-8.0.1-00bfb3?style=flat&logo=elastic-stack)](https://www.elastic.co/blog/category/releases)
# Elastic Stack
My ELK stack, referenced by elastic.co website

Elastic Stack, also known as ELK Stack, is a stack that comprises of three popular projects: 

* Elasticsearch
* Logstash
* Kibana
 
and other components such as:
- Beat
- APM

Elastic Stack is used to take data from any source then search by Elasticsearch, analyze and visualize by Kibana. In this repository, I will only go into detail about Elasticsearch and Kibana.

## Contents

1. [Components](#Components)
	* [Elasticsearch](#Elasticsearch)
	* [Kibana](#Kibana)
2. [Setup](#set-up)
	* [Host requirements](#Host-requirements)
	* [Setup steps](#Setup-steps)
3. [Backup and restore](#Backup-and-restore)
	* [Backup](#Backup)
	* [Restore](#Restore)

## Components
### Elasticsearch
### Kibana

## Setup
### Host requirements
* [Docker Engine][docker-install] version **18.06.0** or newer
* [Docker Compose][compose-install] version **1.26.0** or newer (including [Compose V2][compose-v2])
* More than **6 GB** of RAM (because the host consumes lots of memory when runs ELK multi nodes)
### Setup steps
> **Summary**  
> This document will show how to use Docker and Docker Compose to install 3 
> Elasticsearch nodes in 1 cluser, Kibana to visual and management them, and how to backup 
> and restore the database. The way to install is the final, after hours of fixing bugs during the installation.
> The **demo** is at [here](https://www.youtube.com).

In a Ubuntu server, create a directory and move into it:

	
    $ mkdir docker-ELK && cd $_
    
	
Make a `docker-compose.yml`file with the contents from [`docker-compose.yml`](docker-compose.yml)

> **Note**  
> This file is at first referenced from the [docker-compose file][docker-compose-file] of [elastic.co], however I have edited it to fix bugs when build the docker
> compose up. What I have edited is setting up JVM heap size in `environment` tag `- ES_JAVA_OPTS=-Xms750m -Xmx750m` in each Elasticsearch node, which prevents the nodes from exiting.

Also create `.env` file with the contents from [`.env`](.env)

> **Note**  
> This file is at first referenced from the [.env file][.env-file] of [elastic.co], however I have edited it. Beside `password` and `version`, I have also edited 
> `MEM_LIMIT` = **6442450944** bytes, which is more than **6 GB**.

And increase the limit of `mmap` (virtual memory of the host):

	$ sudo sysctl -w vm.max_map_count=524288
	
Finally, at the docker-ELK directory, run the command:
	
	$ docker compose up -d 

Wait for about 3 minutes for the ELK to setup.
> **Note**  
> If there are problems, run `$ docker compose logs -f <service>` to observe the logs and exit code (search google for it). If you meet the **137** exit code, I recommend [this][exit-code-137] .
 
When all 3 nodes are healthy, access the Kibana web UI by opening http://IP-address-of-the-host:5601 in a web browser and use the following (default) credentials to log in:

	user: elastic
	password: abc123 



## Backup and restore
### Backup
	
First, display the list of ELK containers:
	
	$docker ps
	
then access to **each** container with `root`:

	$docker exec -u 0 -it <esearch container id> /bin/bash
	$apt-get update
	
Install the nano text editor:
	
	$apt-get install nano
	
Ctrl – D to quit the container and get access to it again with user elasticsearch:

	docker exec -it <esearch container id> /bin/bash

Create a backup directory:
	
	mkdir backup_repo

Config the `elasticsearch.yml` file to create a path to the backup directory:

	nano config/elasticsearch.yml

Add `path.repo: /usr/share/elasticsearch/backup_repo` to the file 

### Restore:










[elastic.co]: https://elastic.co
[docker-compose-file]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-file
[.env-file]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-env-file
[elk-stack]: https://www.elastic.co/what-is/elk-stack
[xpack]: https://www.elastic.co/what-is/open-x-pack
[paid-features]: https://www.elastic.co/subscriptions
[es-security]: https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html
[trial-license]: https://www.elastic.co/guide/en/elasticsearch/reference/current/license-settings.html
[license-mngmt]: https://www.elastic.co/guide/en/kibana/current/managing-licenses.html
[license-apis]: https://www.elastic.co/guide/en/elasticsearch/reference/current/licensing-apis.html
[exit-code-137]: https://stackoverflow.com/questions/62006956/elasticsearch-multi-node-cluster-one-node-always-fails-with-docker-compose
[elastdocker]: https://github.com/sherifabdlnaby/elastdocker

[docker-install]: https://docs.docker.com/get-docker/
[compose-install]: https://docs.docker.com/compose/install/
[compose-v2]: https://docs.docker.com/compose/cli-command/
[linux-postinstall]: https://docs.docker.com/engine/install/linux-postinstall/

[bootstrap-checks]: https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html
[es-sys-config]: https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html
[es-heap]: https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-size-settings

[win-filesharing]: https://docs.docker.com/desktop/windows/#file-sharing
[mac-filesharing]: https://docs.docker.com/desktop/mac/#file-sharing

[builtin-users]: https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
[ls-monitoring]: https://www.elastic.co/guide/en/logstash/current/monitoring-with-metricbeat.html
[sec-cluster]: https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-cluster.html

[connect-kibana]: https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html
[index-pattern]: https://www.elastic.co/guide/en/kibana/current/index-patterns.html

[config-es]: ./elasticsearch/config/elasticsearch.yml
[config-kbn]: ./kibana/config/kibana.yml
[config-ls]: ./logstash/config/logstash.yml

[es-docker]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
[kbn-docker]: https://www.elastic.co/guide/en/kibana/current/docker.html
[ls-docker]: https://www.elastic.co/guide/en/logstash/current/docker-config.html

[upgrade]: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html
