+++
date = "2017-06-23"
draft = true
title = "Creating a binary debian package"
slug = "creating-a-binary-debian-package"
tags = ["debian", "sysadmin"]
author = "Miguel de la Cruz"
+++

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
