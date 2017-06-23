+++
date = "2015-11-15"
draft = false
title = "Encrypt a USB drive with LUKS"
slug = "encrypt-a-usb-drive-with-luks"
tags = ["security", "encryption"]
author = "Miguel de la Cruz"
+++

One of the best options we have to encrypt a USB is to use the
[Linux Unified Key Setup](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) or **LUKS**. Through this process, we
will format and generate a completely encrypted partition that will require a password to be accesed.

### Requirements

The only requirement is the `cryptsetup` program, that can be installed through your favourite distribution's package
manager. In the case of Debian/Ubuntu, you just need to run

```sh
apt install cryptsetup
```

### Formatting the USB

Through all this steps we will supose that the drive is identified into the system with `/dev/sdb`. You need to find
which name the system gave to your USB drive and use it instead.

First thing we need to do is to format the device itself with `cryptsetup`.

```sh
cryptsetup -y -v luksFormat /dev/sdb
```

The next step will be to initialize the volume. Each time we open the volume, the system will create a reference to it
in the `/dev/mapper/` directory. The third parameter on the following snippet is the *mapper* that the device will have
when opened.

```sh
cryptsetup luksOpen /dev/sdb my-encripted-disk
```

The first time we run this command, `cryptsetup` will ask for a *passphrase* for the volume. After introducing one, it
will create a `/dev/mapper/my-encrypted-disk` mapper, that represents our volume unencrypted.

We can check its state using

```sh
cryptsetup -v status my-encripted-disk
```

Last thing we need to do is to format the partition using our favourite format. In this example, we will use plain
`ext4`

```sh
mkfs.ext4 /dev/mapper/my-encrypted-disk
```

### Mount the volume

When the mapping is active, we can mount the volume as a normal partition

```sh
mount /dev/mapper/my-encrypted-disk /mnt
```

### Unmount and extract the device

To extract the device, first we need to unmount the volume, then close the device through `cryptsetup` and then we can
safely extract it

```sh
umount /mnt
cryptsetup luksClose my-encrypted-disk
```

### Summing up

It is not difficult to encrypt and use a USB drive to securely carry around our information. It just requires one step
more than any other drive to be mounted into the system and ready to use.

Remember that the steps are:

1.  Use `cryptsetup` and **luksOpen** to create a mapper.
2.  `mount` the mapper as a normal volume.

Graphical file managers as `nautilus` will even automatically detect and manage this kind of devices, easing the pain of
having to do all the mounting manually.
