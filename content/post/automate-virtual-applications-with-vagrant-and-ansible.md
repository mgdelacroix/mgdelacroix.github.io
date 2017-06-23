+++
date = "2016-01-26"
draft = true
title = "Automate virtual applications with Vagrant and Ansible"
slug = "automate-virtual-applications-with-vagrant-and-ansible"
tags = ["vagrant", "ansible", "sysadmin", "devops"]
image = "images/automate-virtual-applications-with-vagrant-and-ansible_vagrant.png"
share = false
author = "mgdelacroix"
+++

At [Kaleidos](http://kaleidos.net), we use out own [GitLab](https://about.gitlab.com/) instance to manage and centralize
our project's codebase. GitLab release a new version
[each month](https://about.gitlab.com/2015/12/17/gitlab-release-process/), so we sysadmins have our own schedule to
update our instance. As we basically depend on the tool for our collaborative work, we cannot afford any problems, so
each time GitLab releases a version, we update to the previous one after testing that the update will go smoothly.

To test the update, we usually created a new VM with our debian version, installed our current GitLab version using
[the Omnibus package](https://about.gitlab.com/downloads/), load our current production backup, update the application
package to the desired version and test the result. The whole process is easy, but time consuming.

This workflow can be easily automated with [Vagrant](https://www.vagrantup.com/) and
[Ansible](http://www.ansible.com/). These tools will allow us to create a virtual machine and provision it with the
GitLab version we want, so we can load the backup and update the version quickly.

### Vagrant

Vagrant is a command line tool to create and manage development environments. It supports
[multiple providers](https://www.vagrantup.com/docs/getting-started/providers.html), like `aws`, `vmware` and `docker`,
but we are going to use the default one, `VirtualBox`. As we like to play with the different options that GitLab
includes with each release, we've like better (and closer to the real environment) to use a whole VM than a simple,
single-process bind docker container.

To install Vagrant and its requirements in debian, we can run:

{{< highlight sh >}}
apt-get install vagrant virtualbox
{{< /highlight >}}

Vagrant uses a file written in ruby called `Vagrantfile` to describe the VMs that we are going to use and their
properties. This is the `Vagrantfile` we use to create a debian based VM.

{{< highlight ruby >}}
Vagrant.configure(2) do |config|
  config.vm.box = "debian/jessie64"
  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "gitlab.yml"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048
  end
end
{{< /highlight >}}

The `config.vm.box` property indicates the box that we will use as our VM base. You can find more boxes in the
[Atlas Directory](https://atlas.hashicorp.com/boxes/search). This `Vagrantfile` is almost self-explanatory, you can find
a complete reference to all the parameters used at the
[Vagrant documentation](https://www.vagrantup.com/docs/vagrantfile/).

These are the basic Vagrant commands. All of them should be executed in the directory where we created the `Vagrantfile`:

- `vagrant up`: starts the virtual machines described in the `Vagrantfile`. If the machine doesn't exists, creates
  it. If

mount /vagrant

{{< highlight yaml >}}
---
- name: Install gitlab in a brand new VM
  hosts: all
  vars:
    debian_version: jessie  # can be wheezy too
    gitlab_version: gitlab-ce_8.0.4-ce.1_amd64
    temp_package_path: /tmp/gitlab.deb
  tasks:
    - name: Install and configure the necessary dependencies
      become: yes
      apt: name={{ item }} state=latest update_cache=yes
      with_items:
        - curl
        - openssh-server
        - ca-certificates
        - postfix

    - name: Download gitlab package
      get_url:
        dest: "{{ temp_package_path }}"
        url: https://packages.gitlab.com/gitlab/gitlab-ce/packages/debian/{{ debian_version }}/{{ gitlab_version }}.deb/download

    - name: Install the gitlab package
      become: yes
      apt: deb={{ temp_package_path }} state=present

    - name: Configure and start gitlab
      become: yes
      command: gitlab-ctl reconfigure
{{< /highlight >}}
