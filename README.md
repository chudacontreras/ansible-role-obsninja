# Ansible Role: OBS Ninja

**Table Of Contents**
- [Ansible Role: OBS Ninja](#ansible-role-obs-ninja)
  - [Overview](#overview)
    - [Inspiration](#inspiration)
    - [About OBS Ninja](#about-obs-ninja)
  - [Tasks performed](#tasks-performed)
  - [Requeriments](#requeriments)
  - [Role Variables](#role-variables)
    - [Ec2 Role](#ec2-role)
    - [Obsninja Role](#obsninja-role)
  - [Playbook Example](#playbook-example)
  - [Caveats](#caveats)
  - [MIT License](#mit-license)

## Overview

This project automates the installation and configure a stand alone OBS Ninja server on Ubuntu 20.04 batteries included.

This project is work in progress and is open to collaborators.

### Inspiration

Inspired on [The AWS Blogger (Jon Myer)](https://www.youtube.com/c/TheAWSBlogger/about) video [OBS Ninja Install on AWS | OBS.Ninja on AWS](https://www.youtube.com/watch?v=8sDMwBIlgwE&list=PL8VJWj2-XLFpFu3G35Hdm1nKZ2xn9_0_8&index=2)

### About OBS Ninja

OBS.Ninja is a software that uses peer-to-peer technology to bring remote cameras into OBS, created by [Steven Seguin](https://github.com/steveseguin) [@xyster](https://twitter.com/xyster)

[Official OBS.Ninja repository](https://github.com/steveseguin/obsninja)

## Tasks performed

* Infrastructure
  * create security groups for your instance
  * Creates your Ec2 instance to install obs ninja
  * Creates the dns record set pointing to your instance ip.
* Obs Instalation
  * Apply the latest updates and security patches to the instance Operating system.
  * Install all dependencies needed
  * Creates an SSL certificate, installs and configure it.
  * Download and install Obs Ninja from sources

## Requeriments

* aws cli
* ansible
* ansible galaxy collections
  * amazon.aws
  * community.aws

## Role Variables

This Playbook contains two Roles:

* Ec2: This role provision an ec2 instance, security groups, and dns recordsets for your OBS ninja, so it takes care of everything for you.
* obsninja: This Role installs and configure Obs on the instance provided by the Ec2 role or can work stand alone to do the same with instance with ubuntu 20.04 already created you have to provide the inventory.

### Ec2 Role

`profile`: Indicates to ansible which aws cli profile to be used to authenticate with the aws api on you behalf.

`region`: Selects the aws desired reg`  ion where the role will deploy the infrastructure form OBS ninja.

`instance_name`: Sets the instance Name tag on the ec2 console which helps to identify the OBS instance on your AWS Ec2 web console.

`ami_id`: selects the base image to spin up the instance where Obs ninja will be installed by our ansible role, considering this role is in a early stage only supports.

`key_name`: selects the key that ansible will use to authenticate with your Ec2 instance.

`instance_type`: Selects the Ec2 instance type.

`volume_size`: This is the volume size for your instance main storage.

`vpc_id`: Indicates the VPC where your instance will be placed.

`subnet_id`: indicates the specific subnet of your vpc where the instance will be placed.

`server_name`: indicates the hostname that will be used to identify you instance, this will be used to create the dns record for your instance

`domain_name`: This setting selects which Route53 hosted zone will be used to create the dns record set for the instance. This hosted zone must pre-exists and should be operational, other wise the obs ninja role will be unable to validate the ssl certificates and will fail.

_Variables example:_

```ansible
vars:
  ec2:
    profile: "default"
    region: "us-west-2"
    instance_name: "Obs Ninja"
    ami_id: "ami-0ca5c3bd5a268e7db"
    key_name: "myself-key"
    instance_type: "t3.micro"
    volume_size: "10"
    vpc_id: "vpc-xxxxxxxxxxxx"
    subnet_id: "subnet-xxxxxxxxxxxx"
    server_name: "ninja"
    domain_name: "yourdomain.com"
 ```

### Obsninja Role

`server_name`: this is the server FQDN that will be used by nginx and also by CertBot to generate the ssl certificate for your in-transit encryption.

`site_root`: location at the server where you want to deploy the obsninja code base.

`admin_email`: The email is required to issue the ssl certificate by Let's Encrypt via CertBot, this field is important cause your cert needs to be associated with you email address.

```ansible
vars:
  obsninja:
    server_name: "ninja.yourdomain.com"
    site_root: "/var/www/obs"
    admin_email: me@yourdomain.com
```

## Playbook Example

This is a full example, please update it with your valid configurations:

* _ec2_
  * profile
  * key_name
  * vpc_id
  * subnet_id
  * server_name
  * domain_name
* _obsninja_
  * server_name
  * admin_email

```ansible
---
- hosts: localhost
  connection: local
  gather_facts: true

  vars:
    ec2:
      profile: "myself"
      region: "us-west-2"
      instance_name: "Obs Ninja"
      ami_id: "ami-0ca5c3bd5a268e7db"
      key_name: "myself"
      instance_type: "t3.micro"
      volume_size: "10"
      vpc_id: "vpc-xxxxxxxxxxxx"
      subnet_id: "subnet-xxxxxxxxxxxx"
      server_name: "ninja"
      domain_name: "yourdomain.com"

  roles:
    - ec2

# Stage two
- hosts: launched
  become: yes
  gather_facts: true

  vars:
    obsninja:
      server_name: "ninja.yourdomain.com"
      site_root: "/var/www/obs"
      admin_email: me@yourdomain.com
  roles:
    - obsninja

```

## Caveats

This is an open source software project provided as is to demonstrate how easily is deploy a OBS Ninja server on AWS almost without manual steps other than the DNS zone.

This make you incur in expenses on your aws account that, please be conscious about them when you select the instance type and storage size, another cost to consider is the traffic in and out.

My advice is sets costs alerts in your aws account in order to prevent spend more than you expect.

## MIT License

Copyright (c) 2021 Damian Antonio Gitto Olguin

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND  NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
