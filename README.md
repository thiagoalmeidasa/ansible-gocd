GOCD Challenge
==============

## What is it?

The goal of this project is install and do some configuration in a new
instance of [GOCD](https://www.gocd.io/) running in a Debian 8.

## Features

What is happening here?

* Some basic configurations like locale, ntp, timezone and some iptables input
  block are done.
* A fresh installation of GOCD from official deb repository. 
* Install some GOCD plugins for:
	- Elastic agents with docker;
	- Email notifications using `STARTTLS` instead of `SMTPS`;
	- Tasks with script executor;
	- Debian repository poller;
* Configure GOCD auth with htpasswd file;
* Install Docker Engine for elastic agent purpose;
* Build a single docker image from official maven image with auto register
  agent;

## How it works?

For a complete running instruction see the [INSTALL.md](INSTALL.md).

### Quick start

Configure your `inventory/hosts`, changing the `ansible_host=192.168.0.26` to
your IP and do a simple:
```bash
ansible-playbook --ask-vault-pass -i inventory/ main.yml
```
You will need to answer the _ansible vault_ password, for this example will be
`password-example`.

The new server will be configured with the features listed above and running
on `http://YOUR_IP:8153` and `https://YOUR_IP:8154` with self-signed
certificate.

Now you can configure your pipelines with email notifications and docker
elastic agents.

For email notifications check: https://github.com/gocd-contrib/email-notifier

For docker elastic agents check:
https://github.com/gocd-contrib/docker-elastic-agents#usage-instructions
