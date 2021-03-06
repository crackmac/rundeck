Rundeck Cookbook
================
Installs and configures a Rundeck 2.0 server with Chef integration via the chef-rundeck.gem.  Projects in rundeck can be dynamiclly configured via data bag items using search.  Linux and Windows client nodes are supported.  The cookbook has optional support for Active Directory and LDAP.

[![Build Status](https://travis-ci.org/Webtrends/rundeck.png?branch=master)](https://travis-ci.org/Webtrends/rundeck) [![Code Climate](https://codeclimate.com/github/Webtrends/rundeck.png)](https://codeclimate.com/github/Webtrends/rundeck)

Requirements
------------
### Chef
Chef version 0.10.10+ and Ohai 0.6.12+ are required.

Because of the heavy use of search, this recipe will not work with Chef Solo, as it cannot do any searches without a server.

This cookbook relies on multiple data bags. See __Data Bag__ below.

### Platform
* Debian 6
* Ubuntu 12.04
* Windows 7 Enterprise (managed node)
* Windows 2008 R2 (managed node)

**Notes**: This cookbook has been tested on the listed platforms. It may work on other platforms with or without modification.


### Cookbooks
* Java
* Apache2
* Sudo
* Runit


Attributes
----------
### default
Linux default attributes for all rundeck managed nodes and server

* `node['rundeck']['user']` - Rundeck username (linux), default 'rundeck'
* `node['rundeck']['user_home']` - Rundeck user home directory (linux), default '/home/rundeck'

Windows default attributes for all rundeck managed nodes

* `node['rundeck']['windows']['user']` - Windows user to create, default 'rundeck'
* `node['rundeck']['windows']['group']` - Windows user group to add the 'rundeck' user to, default 'Administrators'

### chef-rundeck
Chef rundeck integration service attributes

* `node['rundeck']['chef_config']` - Chef-Rundeck client configuration, default '/etc/chef/rundeck.rb'

* `node['rundeck']['chef_rundeck_url']` - Chef-Rundeck URL, default 'http://chef.hostdomain:9980'
* `node['rundeck']['chef_rundeck_port']` - Chef-Rundeck binds to port, default '9980'
* `node['rundeck']['chef_rundeck_host']` - Chef-Rundeck binds to address, default '0.0.0.0' 

* `node['rundeck']['chef_webui_url']` - Chef Server Web UI URL, default 'https://chef.hostdomain.com'
* `node['rundeck']['chef_url']` - Chef Server API URL, default 'https://chef.hostdomain.com'
* `node['rundeck']['project_config']` - Generated project configuration from data bags, default '/etc/chef/chef-rundeck.json'
* `node['rundeck']['chef_rundeck_gem']` - Use a custom version of the chef-rundeck gem (eg. local version), default 'nil' uses the gem repo by default

### server
Attributes that configure and manage the installation of the Rundeck server

* `node['rundeck']['configdir']` - Configuration direcotry, default '/etc/rundeck'
* `node['rundeck']['basedir']` - Rundeck installation directory, default '/var/lib/rundeck'
* `node['rundeck']['datadir']` - Rundeck project directory, default '/var/rundeck'
* `node['rundeck']['deb']` - Package file name to install, used in the building of the URL
* `node['rundeck']['url']` - URL for the deb file to download and install 
* `node['rundeck']['checksum']` - Checksum for the deb
* `node['rundeck']['jaas']` - Use built in internal realms.properties file, (options 'activedirectory', default 'internal')
* `node['rundeck']['default_role']` - Require users to be a memeber of this role for Rundeck access, default 'user' 
* `node['rundeck']['hostname']` - VIP or server address for the service, default 'rundeck.hostdomain.com'
* `node['rundeck']['email']` - Email address, default 'rundeck@hostdomain.com'

If you want to use encrypted databags for your windows password and/or public/private key pairs generate a secret using:
```bash
	$ openssl rand -base64 512 | tr -d '\r\n' > rundeck_secret
```
Distrubute to all sytems that will work with rundeck via a recipe and set the path to that file in the following attribute
* `node['rundeck']['secret_file']` - default 'nil' 

* `node['rundeck']['rdbms']['enable']` - enable RDBMS support, default 'false'
* `node['rundeck']['rdbms']['type']` - database type, default 'mysql'

Common RDBMS Configuration
* `node['rundeck']['rdbms']['location']` - RDBMS server name
* `node['rundeck']['rdbms']['dbname']` - database name, default 'rundeckdb'
* `node['rundeck']['rdbms']['dbuser']` - database username, default 'rundeckdb'
* `node['rundeck']['rdbms']['dbpassword']` - database password
* `node['rundeck']['rdbms']['port']` - database port number, default '3306'

Oracle RDBMS Configuration 
* `node['rundeck']['rdbms']['dialect']` - hibernate database dialect, default 'Oracle10gDialect'

Windows Attributes 
* `node['rundeck']['windows']['winrm_auth_type']` - winrm authentication type (options 'basic' or 'kerberos', default: 'basic')
* `node['rundeck']['windows']['winrm_cert_trust']` - winrm SSL security (options 'all', 'self-signed', 'default' (trusted certs only), default: 'all')
* `node['rundeck']['windows']['winrm_hostname_trust']` - winrm hostname security (options 'all', 'strict', 'browser-compatible', default: 'all')

Active Directory/LDAP Attributes
* `node['rundeck']['ldap']['provider']` - LDAP server to connect 
* `node['rundeck']['ldap']['binddn']` - LDAP bind DN
* `node['rundeck']['ldap']['bindpwd']` - LDAP bind password
* `node['rundeck']['ldap']['userbasedn']` - LDAP base user DN search
* `node['rundeck']['ldap']['rolebasedn']` - LDAP base role DN search


Recipes
-------
### default
Includes the `rundeck::node_unix` or `rundeck::node_windows` (depedning on platform) recipe to configure rundeck access on the node.

### node_unix
Configures node the for rundeck access.  Creates the user specified in `node['rundeck']['user']` and manages all SSH keys (see __Data Bag__ below) in the home directory `node['rundeck']['user_home']`. Default for Debian / Ubuntu systems.

### node_windows
Configures node the for rundeck access. Creates the user specified in `node['rundeck']['windows']['user']` and manage the correct group memberships (`node['rundeck']['windows']['group']`) and passwords (see __Data Bag__ below). Default for Microsoft Windows sytems.

### chef-rundeck
Installs the chef-rundeck sinatra application which allows integration with a Chef server.  This recipe can be used on any node including the chef server itself.

### server
Includes `rundeck::default`

The server recipe sets up Apache as the web front end by default.

The recipe does the following:

1. Determine if encyrpted data bags are in use
2. Install rundeck and dependent packages required for the server
3. Sets up configuration directories
4. Create SSH keys from data bag
5. Configure winRM if needed
6. Ensure 'rundeck' user owns the project directory
7. Configures and enables the Rundeck web UI via Apache
8. Starts the Rundeck server service
9. Configure and register Rundeck projects based on the data bag entries 


Data Bags
---------
### Rundeck
Create a `rundeck` data bag that will contain the secrets that will be used to log into the rundeck managed nodes. The data bag can be encyrpted via a secret, if using a encrypted data bag the secret file must be avaiable on each of the managed nodes. Example rundeck data bag item:

```javascript
{
  "id": "secure",
  "description": "Rundeck requires credentals to execute on remote nodes",
  "private_key": "-----BEGIN RSA PRIVATE KEY-----\nMIIEogIBAAKCAQEAt3iZzG ..... -----END RSA PRIVATE KEY-----",
  "public_key": "ssh-rsa AAAAB3NzaC1yc2EAAAA ......... f3OC9Jxe/VcFmtelcmQ== rundeck keys",
  "windows_password": "<plain text password>",
  "chef_rundeck_pem": "-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAx5t2uL0kAD2 ..... qTzvcb1u87qWh7rnRlHUeDQ+nI7ZBFgJK\n-----END RSA PRIVATE KEY-----"
}
```

Generate a public and private key for rundeck to manage nodes via SSH.
```bash
	$ ssh-keygen -N '' -q -f /tmp/rundeck_rsa
	$ cat /tmp/rundeck_rsa | awk '{ printf "%s\\n", $0 }'
	-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAoVVcdyhqZYFfUP/E4hFeRotgE0LBolyWPeDifbOMEK9zRCUx\niwTLAiZlmGRCUMytaslIQ17
	9zU7WM2fIidWbAxyy8L7N/fadLcL2B6HtKMOCcHW/\nXntMplPA8SKM1bjZ81CG1cd+JGP7knHZ07anvIUBlgT3DbwzDEwuAnmyvuqC7RBp\nE1XCmGqNUANQt
	e36+f7SL7GSv8V1H+xeANWM6Y83MEI8hvN0nsWLvEjVZifsyI/v\njtPDV0VRetr+GpyK4ir1naNIG4aRHPnteqzuLX2mmFOFbvRalLE3Gq30qj4/B5Qz\n0wR
	28i7rJZ4Z1K0CO7SDrQeh0TO3pUB2qlmNvwIDAQABAoIBAG2zkHtJ1QcOcFSw\nhhy+eJ95WCvgobAYSuTqjLeypdQWqUc2DzkbWjstBroXumwcsPLCyUteRTA
	colQ0\nBs2KnKwCEL7Yz1MYJQqf3hGUjqHAR2rW9fh12Mnke3a76o3M8w6avASTcPensMGE\nfvyR3/61Zj2vRJpnVULQbhyqyds8oMRNzpvxKM+ogixseJvu3
	5G+Rt4/U2QyBBlr\nkompgPxphKqiilp4J3bCTEVtGbgWYgVn/tleorvA+KqYeAm9thOqItMwKrqtbBon\ndVmJadeqjIHVTx+kiXfKqb9h/685MKcuqbUUYmY
	iPs726ToiB+921OzvwmCuTZjx\nSHDJ2CECgYEAzPxcEe9KCJ6aE5st80qQltvVLwcKsc/YlDsLLJtSMwqQXqHnPMwB\n6/HO460t5/zAIO9hUG6waVq3H54DC
	4lVdvFlta8AXUBEo8ycF4x4024c3H+YmsXf\nIuA5sESbMdgKrgceR3mwatsqYjIM6EEMI0qA2pN4m6EklQxiFmNR/NECgYEAyXvp\ncdZ3HPUoyg674xh+veX
	XvNuC4PxlCE5HdETat9ZUYjnyn74N320uhfXPKm+072dD\n3Sp7QYYCj/CiF+I6MZHDim7aFAkDVBsZQ6pR7zVNc61f69Up6j9iR6CWCD/+kRX4\n/pEY22mMO
	vRrQq2mei74SvaR1lZ5wTZEiYYZRY8CgYAZqECx8fyXRZrNd2/x8tRU\nPaHaaAw7o2NdcmJ8q9hHETxuy98QqgxXhwW5U7TaQ7WcqbnJgoFMPpGLQJDrAb6T\
	ny7VKX2QxR9kPk426GNgKxs6P/tyQCtJaICy4Vm4CeCMmEzgEBERDq7kLX25kJ7go\nNqwYL7s555qXmVwxpy7c8QKBgF3vESTrkdjES2H4gIwdrWknMO9xf5E
	Y2pmGtTV1\nrGqs1+Z7kav71UfnBRubQBxOvBIpGLCRz6j6q1MkIs3zwKG/jWSKzc0tbonVoG+1\nhkF5nkRh/iha1xHIvy8ZpRjvjOVjUxSL3QTeLmyF60PI5
	aZtI4D/d3pwEo+Ll2Ru\nSnXtAoGBAJ105hXm36CGUKDoWN+uKyZVeKf9t4WP02G1pYsJ8GwxxSmoR+aIFoxo\nZ2POfSjybf3hZUMYO9J6jCVMrU05hwPVkwl
	W3zEq86o2+hgjwYBJQodBF8H1FOsa\nGjBkfxSrnrze8e6EYC5GV35bY+/tGsxYO/cHUvQXiMAZZIf/dGQK\n-----END RSA PRIVATE KEY-----
```
Copy the line returned and place in the data bag item as the `private_key`.

```bash
	$ cat /tmp/rundeck_rsa.pub | awk '{ printf "%s\\n", $0 }'
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQChVVx3KGplgV9Q/8TiEV5Gi2ATQsGiXJY94OJ9s4wQr3NEJTGLBMsCJmWYZEJQzK1qyUhDXv3NTtYzZ8iJ1Z
	sDHLLwvs399p0twvYHoe0ow4Jwdb9ee0ymU8DxIozVuNnzUIbVx34kY/uScdnTtqe8hQGWBPcNvDMMTC4CebK+6oLtEGkTVcKYao1QA1C17fr5/tIvsZK/xXUf
	7F4A1YzpjzcwQjyG83SexYu8SNVmJ+zIj++O08NXRVF62v4anIriKvWdo0gbhpEc+e16rO4tfaaYU4Vu9FqUsTcarfSqPj8HlDPTBHbyLuslnhnUrQI7tIOtB6
	HRM7elQHaqWY2/ rundeck@rundeckserver
```
Copy the line returned and place in the data bag item as the `public_key`.

Use knife to create a new client for Chef-Rundeck integration.
```bash
	$ knife client create -a chef-rundeck -f /tmp/chef-rundeck
	$ cat /tmp/chef-rundeck | awk '{ printf "%s\\n", $0 }'
	-----BEGIN RSA PRIVATE KEY-----\nMIIEowIBAAKCAQEAunxd79sbk2RLP6NRFUCf7ptPuSlhTmqPlDXJPcxjCStUoVbX\n9lcIsVH8FenscWtwReqw7ca
	A5mthm3JKke3Ux1eYfwdgBLICdgBUfDox53nuhIg0\nuUcTfODgMtef2+OxEx2Fu0fxydqFM6bIeLi8+REDQ9I/ew5Serreg3Kg+SMWYjkG\ncW5fB1OTaVz6/
	Xk9xXWy/pa7rH/Tnq3JEgpG34a6w1+j+NBlpqjS/RmtXUaJQjCb\nrrektQw0P+gfW2+jd+z46DyZSFPT0KYorPIugujdzzJ/Tbl/DBGj0MdA2r2ZDsNX\nHMh
	KV0Qqy5s5f3PsHr3+JWC3eyZBFtlh6wzVKwIDAQABAoIBAAwSFLp7wjMuILjD\nx3HKtw9ouiZQCV5cA2MigB4h5p8nUNkIl/338DYaCmkYtRc6TxAXetBJMvq
	3JKA9\nK5p6fHVStCo0vgBPzVz59H39/lDvUYL+lfsQILDKlXh1AIHpIQMNvCQ9KedY35pS\nR1OZEZJFiaKQL0+1w5zyD4kOmGDHyk/QTekq7HRN2Z3FXB7Ez
	C+44P4NJlM8oK1G\nqEouTGMDoy8bK38Zfd3m6ym2sWBNoSfQe3jufNbWeOW91unSrs74jz+8l0yTn6J/\n9zuDtOvbUfWJubcHIcNiUG5+x7OIIpecaOOwrgk
	Wdhxmf0hkNBQaPD+cBaM82NhK\n+dpvRsECgYEA5SfVo5s7syqFOWVa2YjCVYyBFxS6BG7t51RVjRAPyMhV6L/gBnNy\nIJBDKc7XTcWCdCQZvWOEKHqLtMVOp
	5vr7lydSukXqcvAVX6hQ7AYuymjjKW2di8K\nqPbvMn/1tEkCZcEPC6Eik7aNc78KYbJGo9bV/W+H0woO75Fv5JeWvvECgYEA0FTn\nkRiIBwk0WJcUe7rnu2m
	ITMzi2UEDQ3ClYvFJZQblIYI1tj9ssf5UsutH3BfD/pSi\n67oBDCQX/VqR8eolKbi9Qo7Ix9PI8yX9ELYvPYs5ntbyPmw9kvGGJm8pSNbHvthy\nPjNBME4gX
	7uKdxGkvqq9IkYQlFfWBBXHBYI5TdsCgYB+I6ZC79E194LsLDGNKu2m\nP7hTZzJZ/GHyg4awJpY5tKUtgGklw+ifqil+WwBDLCR6H+EXUi9ORN6gPDfmpTqC\
	ns/JVaOeArMqLhS/p3YZPiEUhx5ofhhd9GKhkiPFMMyAhuNq6URGCc+t7Oj7RtluS\nFlEmt3zxm0jLcKhCEXuGUQKBgQCB4l1Y9b1g/ZkYHmET3uw4yMvEbfy
	ETGcXdbR2\n4k3K4aia4o5QKGzA7/qobc2oZ1y3bL3CT331rs8SEpRpCXzP7TB5vYFqLBzdkvKa\np6r+KL3szL/MsTkWUuQ7NBS+J8HytwlKxDPBRQQkC02Bf
	IuEn/g41QvjIHv6ogUp\n5w2I/wKBgG28Z5IAWXa3g5hPe4D2kfOVp+fIAsvMqosc74QLd61lRSX25YU1CgVG\nB+Jgt4trIHPnPqQ5rC8PuHI5khcRObLHr48
	yCBfa+Xy7nF/HoPuULDzqIjJccHuJ\ncZC8J0MnQaZvJolodhcCYMK2B6UtRpwmn96oNKsbBBT5WU2f8dEI\n-----END RSA PRIVATE KEY-----\n
```	
Copy the line returned and place in the data bag item as the `public_key`.

Set a windows password if managing windows systems, the password needs to be in plain text.  (see the encryption options for the rundeck data bag)

### Encrypted Data bag - Rundeck
When using `node['rundeck']['secret_file']` you will need to create a secret file for the encryption.  Make sure the 'rundeck_secret' file is available on all nodes managed by rundeck.

Generate a secret file with openssl:
```bash
    $ openssl rand -base64 512 | tr -d '\r\n' > /tmp/rundeck_secret
```
Generate the encrypted data bag:
```bash
    $ knife data bag create rundeck secure --secret-file /tmp/rundeck_secret
```

### Rundeck Projects
Create a `rundeck_projects` data bag that will contain the projects and search strings for the rundeck managed nodes to include by project. Example `rundeck_projects` data bag item:

```javascript
{
  "id": "dev-systems",
  "hostname": "ipaddress",
  "username": "rundeck",
  "pattern": "chef_environment:dev1 OR chef_environment:dev2",
  "description": "These instances are tied to the dev-systems project in Rundeck.",
  "chef_rundeck_url" : "Optional: URL for the chef-rundeck integration endpoint"
}
```
 * `hostname` - attribute in the data bag item json is used when rundeck try to connect to the node (`fqdn` is the default)
 * `username` - attribute is the user to authenticate to the node with when rundeck connects
 * `pattern` - attribute is a search query for nodes to include in to the project in rundeck. 
 * `chef_rundeck_url` - optional attribute is a URL to locate the resource project, if not provided `node['chef_rundeck_url']` will be used. 


Rundeck Role ACL Policy
------------------
A default role acl policy is supported out of the box.  You can add new acl policy files in to the configuration directory (`node['rundeck']['configdir']`)

[Rundeck role acl policy definitions](http://rundeck.org/docs/administration/role-based-access-control.html).  



License & Authors
-----------------
- Author:: Peter Crossley <peter.crossley@webtrends.com>

```text
Copyright 2014, Webtrends Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
