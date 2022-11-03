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

  1. In a Ubuntu server, create a directory and move into it:

	
    $ mkdir docker-ELK && cd $_
    
	
  2. Make a `docker-compose.yml`file with the contents from [`docker-compose.yml`](docker-compose.yml)

   > **Note**  
   > This file is at first referenced from the [docker-compose file][docker-compose-file] of [elastic.co], however I have edited it to fix bugs when build the 
   > docker
   > compose up. What I have edited is setting up JVM heap size in `environment` tag `- ES_JAVA_OPTS=-Xms750m -Xmx750m` in each Elasticsearch node, which
   > prevents the nodes from exiting.

  3. Create `.env` file with the contents from [`.env`](.env)

   > **Note**  
   > This file is at first referenced from the [.env file][.env-file] of [elastic.co], however I have edited it. Beside `password` and `version`, I have also 
   > edited 
   > `MEM_LIMIT` = **6442450944** bytes, which is more than **6 GB**.

  4. Increase the limit of `mmap` (virtual memory of the host):

	$ sudo sysctl -w vm.max_map_count=524288
	
  5. At the docker-ELK directory, run the command:
	
	$ docker compose up -d 

   Wait for about 3 minutes for the ELK to setup.
   > **Note**  
   > If there are problems, run `$ docker compose logs -f <service>` to observe the logs and exit code (search google for it). If you meet the **137** exit 
   > code, I recommend [this][exit-code-137] .
 
  6. When all 3 nodes are healthy, access the Kibana web UI by opening http://IP-address-of-the-host:5601 in a web browser and use the following (default) credentials to log in:

	user: elastic
	password: abc123 

## Backup and restore
### Backup
	
  1. Display the list of ELK containers:
	
	$docker ps
	
  2. Access to **each** container with `root`:

	$docker exec -u 0 -it <esearch container id> /bin/bash
	$apt-get update
	
  3. Install the `nano` text editor:
	
	$apt-get install nano
	
  4. `Ctrl–D` to quit the container and get access to it again with user elasticsearch:

	docker exec -it <esearch container id> /bin/bash

  5. Create a backup directory:
	
	mkdir backup_repo

  6. Config the `elasticsearch.yml` file to create a path to the backup directory:

	nano config/elasticsearch.yml

  7. Add `path.repo: /usr/share/elasticsearch/backup_repo` to the file. Save the file with `Ctrl-S` and exit with `Ctrl-X`.
  > **Note**  
  > To know the location of directory `backup_repo`, use command `pwd`
  
Do these above steps for **3** elasticsearch node. Then go to the Kibana web UI. 

  1. Go to **Stack Management** -> **Snapshot and Restore** -> **Repositories**

  ![image](https://user-images.githubusercontent.com/93396414/199712256-0c617f51-7ee9-4c2f-a9e7-a7355b53738b.png)

  2. Select **Register repository**. 
  3. Name **Repository name** as `demo`.
  4. At **Repository type**, select **Shared file system** and Next.
  
  ![image](https://user-images.githubusercontent.com/93396414/199716916-612ef447-49b9-434a-8f51-695472d90408.png)
  
  5. At the `File system locatio`, type: `/usr/share/elasticsearch/backup_repo`. Ignore other fields and **Save**.
  
  ![image](https://user-images.githubusercontent.com/93396414/199718410-c2adf821-ec40-4035-8dad-773cedf0d0a8.png)
  
  6. Then move to **Policies** -> **Create policy**
  7. At step 1, **Logistics**, type as your choice and **Next**.
  
  ![image](https://user-images.githubusercontent.com/93396414/199718947-6c19bf78-c41c-46da-a914-644f3e32c3dd.png)
  
  8. At step 2, **Snapshot settings**, in **Data streams and indices**, choose only my index: favor_candy 
   ![image](https://user-images.githubusercontent.com/93396414/199720039-dba0cd14-c73b-409f-bb77-da08e789a58d.png)

   > **Note** 
   > If the **Include global state** show up, then turn on it.
   Then **Next**. 
   9. Step 3 is not important in this document. I only set the Maximum count to 4. Then **Next**.
   
   ![image](https://user-images.githubusercontent.com/93396414/199720548-f2b418f6-6f72-4338-b6b4-39925593b373.png)
   
   10. At step 4, **Review**, check again.
   
   ![image](https://user-images.githubusercontent.com/93396414/199720718-10ef5b4f-f0cd-44a1-9dc5-4748402b2939.png)
   
   If nothing to edit, then **Create policy**.
   11. A pop-up show up, choose **run now**
   
   ![image](https://user-images.githubusercontent.com/93396414/199721156-c801d522-267b-4f82-8c7a-f53d23b0604c.png)
   
   The successful snapshot will look like this:
   
   ![image](https://user-images.githubusercontent.com/93396414/199721328-d5850f3c-a27a-4267-a437-ded58865db18.png)


   
### Restore:

> **Note**  
> Elastic has strict rules about restoring indices. It will not allow to restore system files unless they are deleted. This part will restore the indices that > I have written in `Dev Tools`
























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
