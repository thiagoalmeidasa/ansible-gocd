INSTALL instructions
====================


<!-- vim-markdown-toc GFM -->
* [Introduction](#introduction)
* [Requirements](#requirements)
* [Directory structure](#directory-structure)
* [Running](#running)
* [Customization](#customization)
    * [Adding new users:](#adding-new-users)
    * [Changing GO Server startup options](#changing-go-server-startup-options)
    * [Choosing GOCD plugins](#choosing-gocd-plugins)
    * [Changing the timezone](#changing-the-timezone)
    * [Adding swap](#adding-swap)
* [Configuring Plugins](#configuring-plugins)
    * [Docker Elastic Agent](#docker-elastic-agent)
    * [Email Notifier](#email-notifier)
* [Configuring Elastic Agent Profiles with Docker Plugin](#configuring-elastic-agent-profiles-with-docker-plugin)
* [Some informations about this approach](#some-informations-about-this-approach)

<!-- vim-markdown-toc -->

## Introduction

This project was designed to install GOCD in a instance running Debian 8.

## Requirements

This project requires `Ansible >= 2.3` in the control machine and
`python-simplejson` package on the managed node.  Everything else will be
installed and configured on demand.

## Directory structure


```
playbooks/
├── HISTORY.md
├── INSTALL.md
├── README.md
├── main.yml # Top level playbook.
├── inventory/ # Inventory dir.
│   ├── host_vars/
│   │   └── server.go.local # Enviroment variables for our server.
│   └── hosts # Hosts lists for those playbooks
├── roles/ # Roles base dir
│   ├── common/ # Basic configuration.
│   │   ├── handlers/
│   │   │   ├── main.yml
│   │   │   ├── swap.yml
│   │   │   └── ntp.yml
│   │   ├── defaults/
│   │   │   └── main.yml # Some default vars
│   │   └── tasks/
│   │       ├── main.yml
│   │       ├── iptables.yml
│   │       ├── locale.yml
│   │       ├── ntp.yml
│   │       ├── timezone.yml
│   │       ├── swap.yml
│   │       └── update.yml
│   ├── docker/ # Take care of docker installation
│   │   ├── files/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── tasks/
│   │       └── main.yml
│   └── goserver/ # Take care of goserver installation and configuration
│       ├── files/
│       ├── handlers/
│       │   └── main.yml
│       └── tasks/
│           ├── main.yml
│           ├── go-auth.yml # Configure go auth with htpasswd
│           ├── go-plugins.yml # Install some go plugins
│           └── go-server.yml
└── vars/global.yml
    ├── global.yml
    └── goauth.yml # Configure users for go auth

```

## Running

Configure your `inventory/hosts`, changing the `ansible_host=192.168.0.26` to
your IP and do a simple:
```bash
ansible-playbook --ask-vault-pass -i inventory/ main.yml
```
You will need to answer the _ansible vault_ password, for this example will be
`password-example`.

The new server will be configured with the features listed above and running on
`http://YOUR_IP:8153` and `https://YOUR_IP:8154` with self-signed certificate.

Now you can configure your pipelines with email notifications and docker elastic
agents.

For email notifications check: https://github.com/gocd-contrib/email-notifier

For docker elastic agents check:
https://github.com/gocd-contrib/docker-elastic-agents#usage-instructions

## Customization

Some configurations can be customized changing environment variables in
`inventory/host_vars/`, `vars/global.yml` and `vars/goauth.yml`. Check it out to
see what can be modified.

### Adding new users:

Edit `vars/goauth.yml` with:

```bash
ansible-vault --ask-vault-pass edit vars/goauth.yml
```
You will need to answer the _ansible vault_ password, for this example will be
`password-example`.

The content looks like:

```
---
goauth_users:
  user1:
    name: 'user'
    pass: 'user'
  user2:
    name: 'admin'
    pass: 'admin'

```

To add a new user create a new item like:

```
---
goauth_users:
  user1:
    name: 'user'
    pass: 'user'
  user2:
    name: 'admin'
    pass: 'admin'
  newuser:
    name: 'newuser'
    pass: 'newpassword'

```

### Changing GO Server startup options

To change GO Server startup options change the content in
`inventory/host_vars/server.go.local` from:

```
GO_SERVER_SYSTEM_PROPERTIES: '-Dmail.smtp.starttls.enable=true'

```
to your new value.

This setting modifies server mail configuration was modified to allow you send
email trouhgt STARTTLS instead of SMTPS.

### Choosing GOCD plugins

This project can install those plugins: `docker-agent deb-repo-poller
email-notifier script-executor-task` and you can chose what of they will be
installed on your instance. Just modify the content of
`inventory/host_vars/server.go.local` from:

```
goserver_plugins: 'docker-agent deb-repo-poller email-notifier script-executor-task'
```

letting the plugins do you want to install.

### Changing the timezone

Edit `vars/global.yml`, change the content from:

```
timezone: America/Belem
```

to your new value.

### Adding swap

To create a new swap file you will need to setup some environment variables on
server `host_vars` or `group_vars`.

The available variables with their default values are:

```
swap_path: /swap
swap_size: False
swap_swappiness: False
swap_vfs_cache_pressure: False
swap_use_dd: False
```

Only `swap_size` is mandatory and you can use `MB` or `GB` as unit of value.

## Configuring Plugins

That's some configuration examples for Docker elastic profile and Email
Notifier plugins.

### Docker Elastic Agent

* Go to `admin/plugins` page;
* Click on `Docker Elastic Agent Plugin` configuration button;
* Configure:
    - `Go Server URL` with `https://YOUR_IP/go` (don't worry about the self
      signed certificate, the auto register agent will handle it);
    - `Agent auto-register Timeout (in minutes)` to something between 1-3 min;
    - `Maximum docker containers to run` to something your server can handle;
    - `Docker URI` to `unix:///var/run/docker.sock` or your docker api url. If
      you are using docker unix socket, remember to add the user who is
      running GOCD server (commonly go) to the docker group and restart the
      server.
    - If you use docker http api with tls certificates, use `Docker CA
      Certificate` `Docker Client Key` and `Docker Client Certificate` fields
      to configure the connection.

### Email Notifier

* Go to `admin/plugins` page;

<img src="https://github.com/gocd-contrib/email-notifier/raw/master/images/list-plugin.png" width="400" alt="Email notifier on plugin list">

* Configure the plugin with your mail host information and your
  Pipeline/Stage/State information.

<img src="https://github.com/gocd-contrib/email-notifier/raw/master/images/configure-plugin.png" height="450" alt="Email notifier configuration">

## Configuring Elastic Agent Profiles with Docker Plugin


* Go to `admin/elastic_profiles` page;
* Click in `Add` button;
* For `Id`, use a value that has some meaning about what kind of project can be
  built using this agent, something related to the image being used. Like
`docker-maven-3`;
* Use `cd.go.contrib.elastic-agent.docker` for `Plugin Id`;
* Use your docker image with GOCD elastic agent configuration in `Docker
  image`. [This link](https://github.com/gocd-contrib/docker-elastic-agents)
  could be useful to start building a new image;
* Now go to your pipeline settings and configure each job of each stage the
  value of `Elastic Profile Id` with the same value you used in `Elastic Profile
  Agens` `Id` (on this example we used `docker-maven-3`) and each job will use a
  docker container to execute their tasks.


## Some informations about this approach

* GOCD **script executor plugin** was chosen to make some tasks easier to
  handle.

* Ansible **docker-image** was not used because their dependencies include pip
packages, and I don't want to use pip for just one package.

* Server mail configuration was modified to allow you send email trouhgt
STARTTLS instead of SMTPS.

* The release version of **email-notifier** plugin is too old to be used
sending notifications for specific state of a pipeline, so I needed to build a
new version from master branch of the public repository on github.

* The entire swap configuration is based in: https://github.com/kamaln7/ansible-swapfile
