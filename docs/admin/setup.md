# Setup of ViPLab

## Frontend
There are multiple ways to setup the Frontend. 
If you want to set it up...

* ... in ILIAS: Follow the link specified in the section about the [Integration into ILIAS](integration.md#integration-into-ilias)
* ... in DaRUS: Follow the link specified in the section about the [Integration into DaRUS](integration.md#integration-into-a-research-management-software-like-darus)
* ... in a System of your choice: Take a look at the [Git-Repository](http://example.com/) of the Frontend to find an overview on what you need to use it

!!! error "TODO"
        After publishing the Frontend Repo, change to the correct Git-Repository link

## Websocket API

To set up the Websocket API for getting an interactive session with the ViPLab Service, follow the instructions found in the README of the [viplab-websocket-api](https://github.com/VirtualProgrammingLab/viplab-websocket-api) Git-Repository.

## ECS

You can find Code [here](https://git.freeit.de/ecs4/).

To establish a new ECS, you use [Puppet](https://puppet.com/docs/puppet/6/puppet_overview.html) to install nginx, mysql and set the ECS-Config. 

After that, use the Rake-Script to create the tables in the database:

```
su - ecs -s /bin/bash

cd /srv/ecs/

export GEM_PATH=/usr/lib64/ruby/gems/2.1.0:/srv/ecs/vendor/ruby/2.1.0 

```

And depending on the server: 

```
export RACK_ENV=production
```

or: 

```
export RACK_ENV=development 

bin/rake db:setup
```

As of now (2018-02-16), the [Rake](https://www.rubyguides.com/2019/02/ruby-rake/)-Script creates a table type that is too short for the body. 
Because of that in the mysql-db, you should set it to `MEDIUMTEXT`instead of `TEXT`:
```
ALTER TABLE `messages` CHANGE `body` `body` MEDIUMTEXT CHARACTER SET utf8 COLLATE utf8_unicode_ci NULL DEFAULT NULL;
```

ECS-BD Migration (as Root):

```
cd /srv/ecs/

export GEM_PATH=/usr/lib64/ruby/gems/2.1.0:/srv/ecs/vendor/ruby/2.1.0 

bin/rake db:migrate RAILS_ENV=development
```

!!! error "TODO"
        Anleitung zur Einrichtung eines neuen ECS in Confluence?

## Backends
To find all the requirements to run the ViPLab Backend and start it, follow the following [link](https://github.com/VirtualProgrammingLab/ViPLab-Backend) and carry out the given instructions. 