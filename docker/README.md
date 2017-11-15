# Table of Contents

* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Initial Setup](#initial-setup)
* [Basic Usage](#basic-usage)
* [Creating your own template](#creating-your-own-template)

# Introduction

The docker scripts provided were created to help easily setup services for development and testing.
The scripts were designed to create templates that can be shared and easily customized.

Here's a list of templates that are currently available:
* Databases
  * DB2 Express
  * Mysql
  * PostgreSQL
  * SQL Server
* Search Indexers
  * Elasticsearch
  * Elasticsearch Head
  * Solr
* Web Servers
  * Nginx

# Prerequisites

Before using the docker scripts, make sure you have docker installed.
Link: <https://www.docker.com/community-edition#/download>

# Initial Setup

1\. Clone this repository
```
git clone git@github.com:ericyanlr/liferay-dev-tools.git
```

2\. Add this section to .bash_aliases (or the equivalent on whichever shell you're using).

**Note:** making sure to change /path/to/clone/location to wherever you cloned the repository.
```
LDT_REPO_PATH=/path/to/clone/location

dockertemplate() {
	${LDT_REPO_PATH}/docker/dockertemplate "$@"
}
```

# Basic Usage

1\. To start, we can first get a list of available templates by running the command:
```
dockertemplate -l
```

2\. Let's say we would like to setup a MYSQL server, which is listed as:
```
databases/mysql
```

3\. We can easily generate an alias to add to our .bash_aliases (or the equivalent on whichever shell you're using) by running the command:
```
dockertemplate -a databases/mysql
```

4\. Here is a sample output of the generated alias:
```
# Copy the follwing to your bash aliases:
#
# docker template for MYSQL
# Username: root
# Password: test (configured below via -e MYSQL_ROOT_PASSWORD=test)
# Hostname: localhost
# Port: 3306 (configured below via -p 3306:3306)
dockermysql() {
	/Users/liferay/Liferay/development/repos/liferay-dev-tools/docker/templates/databases/mysql/dockermysql "$@"
}
```
5\. With the generated bash alias added, we can easily start a MYSQL docker container by running the command:
```
dockermysql start
```

6\. MYSQL should now be available on your localhost using port 3306.

# Creating your own template

If you would like to create your own template for adding additional services, you can check out the sample  template blueprint, located in docker/templates/template/blueprint/dockertemplateblueprint

Reference: <https://github.com/ericyanLr/liferay-dev-tools/tree/master/docker/templates/template/blueprint>

With this blueprint, we can easily create additional templates for services.
The main portions that we need to focus on are:
* Renaming the function name: dockertemplateblueprint
* Filling in the following variables:
```
	local dockerimage=""
	local containername=""
	local containercmd=""
	local containeroptions=(
		#"-e foo=bar"
		#"-p 1123:123"
	)
```

For example, if we wanted to add a specific version of MySQL, such as MySQL 5.5, we can set the options to:
```
	local dockerimage="mysql:5.5"
	local containername="ldt_mysql55"
	local containercmd=""
	local containeroptions=(
		"-p 3306:3306"
	)
```

The **dockerimage** that we used can be found in docker hub: <https://hub.docker.com/_/mysql/>

>On the right corner of that page, the suggested command is:
>```docker pull mysql```
>The **mysql** portion is why we add **mysql** to the **dockerimage** variable.

>On the page, you'll also see a section for **Supported Tags**, which lists **5.5** as an option.
>The listed **5.5** tag is why we add **:5.5** to the **dockerimage** variable.

The **containername** that we used can be anything, but we use this notation just for consistency.

The **containeroptions** section is used to set certain flags and options in docker.
>If you look at the docker hub page for mysql: <https://hub.docker.com/_/mysql/>, you'll notice the available options/flags that are made available for this specific docker image.

The **containercmd** is usually optional, but may be needed, depending on the docker image that is being used.
> For example, for MySQL, we can use this portion to set additional settings for our MySQL server.
> Setting the following would make our MySQL server use UTF-8
> ```
> local containercmd="--character-set-server=utf8 --collation-server=utf8_general_ci"
> ```

With the changes above made to this template, you should be able to create a template for MySQL 5.5

For additional guidance, looking at the pre-existing template for MySQL should be helpful: <https://github.com/ericyanLr/liferay-dev-tools/docker/templates/databases/mysql/dockermysql>
