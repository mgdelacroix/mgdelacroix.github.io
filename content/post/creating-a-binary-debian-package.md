+++
date = "2017-06-23"
draft = true
title = "Creating a binary debian package"
slug = "creating-a-binary-debian-package"
tags = ["debian", "sysadmin", "devops"]
author = "Miguel de la Cruz"
toc = true
+++

## Rationale

Imagine you have a complex platform, deployed in a Rack with 10 servers. Ok, better make it two racks, with different layouts.

The platform has different mobile components: a database, a broker, two webservers, one HA proxy, monitoring, email, some shared filesystems and a dozen of microservices. Some of them use custom compiled versions of `ffmpeg` and `libav` to process video and audio.

Imagine you need to deploy and manage all of this, no virtualization allowed. What would be your approach?

First I tried creating plenty of ansible roles, two for each component: one would describe the requirements of the component and the other one would deploy its latest version. This approach worked at first, but I needed to maintain and change those roles every time a component updated its dependencies, and I didn't have any control on the installed files, configuration, or update process.

That's not how software should be delivered. It's easy to think that, as developers, is enough to write a simple `README` file with the requirements of our application and provide with a binary artifact to be deployed on the server. That's fine for simple applications, but it doesn't scale. If your application has to share system, dependencies and resources with other applications, things start to get complicated.

We should think in our software deliverable as an artifact that would describe its dependencies to be integrated and managed by the filesystem. It should have some index mechanism for the system to know which files are owned by it, so they are easy to manage. If we are updating a previous version and some migrations need to be applied or some dependencies modified, all of that should be described in our artifact.

Introducing **debian binary packages**. A debian binary package is a `.deb` file that contains things like:

  - our compiled software
  - a list of the dependencies it needs with their versions
  - scripts to run before and after installing and updating it
  - a mechanism that allows our system to manage every file installed and needed by our software

And the best part of it is that is very easy to create and use.

## About package types

When a maintainer creates a debian package for a software, usually they create two packages, the *source package* and the *binary package*.

The *source package* is born from the tarball that the developer provides, and contains the modifications needed for that software to run in the target Linux distribution (e.g. debian).

The *binary package* is automatically compiled from the source one, and is the one we install in the target machine.

In this article we are going to build only the *binary package*, and we will bypass the compilation process, asuming that we compile the software outside the building process.

### Goal

Needed??

## Anatomy of a debian package

Let's assume that we are going to generate a debian package to distribute our backup script for the platform, called `platform-backup`. This is the structure that we need to create:

```sh
$ tree platform-backup
platform-backup
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   ├── install
│   └── rules
└── platform-backup.sh
```

The `platform-backup` directory contains all the required files to generate our package. In the `debian` directory we will have all the package metadata and a list of the files that we want our package to install. Outside that folder we will have our software artifact, in this case, the `platform-backup.sh` shell script.

### Compat

### Changelog

### Control

### Install

### Rules

## Requirements

### The docker approach

## Step by step




Disclaimer: what this blog post is and what it is not.

- Simple guide to create a .deb package from binary files.
- We are going to ignore the source file and bypass the compilation process. Our starting point are the files that we want to package.
- Perfect for distributing, installing and managing your binaries in a remote machine.
- Compatible with the ansible / docker approach.

## Advantages

- Allows you to manage the dependencies of the package.
- Totally integrated with the O.S.
- You can complain with the FHS and put each file where it belongs.
- It does nothing that a script can do, but it does it better and following the standards.
- You have no excuse not to do it! ;)

## Anatomy of a debian package

- The source package. Why we are not going to use it.
- A little bit about the building process.
- the rules file and the dh

## Requirements

- devscripts and ¿¿build-essential??
- you can put this in a simple dockerfile if you are not using debian/ubuntu

## Steps

- El fichero compat
- El fichero control
- Gestionar el changelog
- El fichero install
- El fichero rules

## Anexo (en inglés?): Dockerfile y script para ejecutarlo
