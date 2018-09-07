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
    + DB2 Express
    + Mysql
    + PostgreSQL
    + SQL Server
* Search Indexers
    + Elasticsearch
    + Elasticsearch6
    + Elasticsearch Head
    + Solr
* Services
    + OpenLDAP
* Web Servers
    + Apache (ProxyPass)
    + Nginx
    + Shibboleth IDP

# Prerequisites

Before using the docker scripts, make sure you have docker installed.
Link: <https://www.docker.com/community-edition#/download>

**Note: For Linux Users**

Docker, by default, requires users to access docker via ```sudo```. But, since these dockerscripts rely on declaring bash functions, they will probably be unaccessible when using ```sudo```. 

One solution is to manage docker as a __non-root__ user.

Docker provides a way to accomplish this: [Manage Docker as a non-root user](https://docs.docker.com/install/linux/linux-postinstall/).

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
	"${LDT_REPO_PATH}"/docker/dockertemplate "$@"
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
# Password: test (configured via -e MYSQL_ROOT_PASSWORD=test)
# Hostname: localhost
# Port: 3306 (configured via -p 3306:3306)
#
# Config file: /Users/liferay/Liferay/development/repos/liferay-dev-tools/docker/templates/databases/mysql/dockermysql
#
dockermysql() {
	/Users/liferay/Liferay/development/repos/liferay-dev-tools/docker/templates/databases/mysql/dockermysql "$@"
}
```

5\. With the generated bash alias added, we can easily start a MYSQL docker container by running the command:

```
dockermysql start
```

6\. MYSQL should now be available on localhost using port 3306.

**Note:** If you already have a MySQL server installed, port 3306 is probably already taken.

The next section will talk about how we can modify the template properties.

# Modifying template properties

When using a template, we may want to modify or add additional properties for the docker container.

An easy way to do this would be to modify the file for the template itself.

This file can be easily found in the generated alias, in the line: **Config file**

Opening the provided file in a text editor, we can find the default properties.

For example, for **databases/mysql**, you should see this snippet in the file:

```
local containeroptions=(
	"-e MYSQL_ROOT_PASSWORD=test"
	"-p 3306:3306"
)
```

If we wanted to change our MySql container's port to 3307 instead of 3306, we could modify the options to the following:

```
local containeroptions=(
	"-e MYSQL_ROOT_PASSWORD=test"
	"-p 3307:3306"
)
```

**Note:** If you already started the container before making your changes, you would have to recreate the container.

To easily recreate the container, we can simply run the following command:

```
dockermysql recreate
```

Starting the newly modified MySQL container should now make it available on localhost using port 3307.

# Advanced Usage

If you are planning to use a container for a long period of time or would like to make sure your custom properties are not overwritten, you may consider adding a more complete alias of the template you are using.

The first step would be to also include the following essential functions to your .bash_aliases (or the equivalent on whichever shell you're using).

```
dockercontainer() {
	"${LDT_REPO_PATH}"/docker/dockercontainer "$@"
}

dockertemplate() {
	"${LDT_REPO_PATH}"/docker/dockertemplate "$@"
}

dockerutil() {
	"${LDT_REPO_PATH}"/docker/dockerutil "$@"
}
```

The next step would be to open the **Config file** of the template you would like to use.

Inside the **Config file**, you should see a section called **Core Function**

For example, for **databases/mysql**, you should see this snippet in the file:

```
#Core Function
dockermysql() {
	# @description create a docker container for MySQL
	# @param $1 available commands listed in usages
	# @param $... additional parameters based on inital command used
	# @output varies, depending on the command used

	local dockerimage="mysql:5.7"
	local containername="ldt_mysql"
	local containercmd="--character-set-server=utf8 --collation-server=utf8_general_ci --max_allowed_packet=50M --innodb-log-file-size=250M"
	local containeroptions=(
		"-e MYSQL_ROOT_PASSWORD=test"
		"-p 3306:3306"
	)

	if [ -n "$1" ]; then
		case "$1" in
			'recreate'|'run'|'start')
				dockercontainer "$1" "${containername}" "${dockerimage}" "${containercmd}" "${containeroptions[@]}";;
			*)
				dockercontainer "$1" "${containername}" "${@:2:${#}}";;
		esac
	else
		dockerutil usage_display ${FUNCNAME[0]} "$(dockertemplate template_usage)"
	fi
}
```

Copy this function to your .bash_aliases (or the equivalent on whichever shell you're using).

Your can also rename the function as needed.

With this new function: **dockermysql()**, you can now easily configure it's properties within your .bash_aliases.

This means that even if the original template were to change (such as when updating the repo), your configurations would not be affected.

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
>
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
