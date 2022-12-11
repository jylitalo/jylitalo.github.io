---
title: "Notes about SSH"
date: 2017-12-05T12:00:00+02:00
categories: ["index", "linux"]
---
Random notes about ssh config and related topics.
<!--more-->
## Making hostnames easier

Openssh reads your _~/.ssh/config/_ before it checks what IP address it should use for connection. This allows you to do things like:

```
Host bastion.company
  Hostname 10.10.10.5
  IdentityFile ~/.ssh/company.pem
  ForwardAgent yes
  User ec2-user
```

or if you wish to by pass bastion:

```
Host _final_destination_
  User _at_bastion_
  ProxyCommand ssh _bastion_host_ netcat %h %p
```

So even if the hostname is not in a DNS or it has some strange name like ec2-10-10-10-5.eu-west-1.compute.amazonaws.com, you can use alternative hostnames with ssh (even if you don't have rights to change _/etc/hosts_ file in your computer).

## Extracting public key from pem file

If you have to give some user ssh access to your servers and they send you PRIVATE KEY PEM file instead of the string that you can attach into their ~/.ssh/authorized_keys_file, you can get it with `ssh-keygen -y -f file.pem`.

If you have to give some user ssh access to your servers and they send you PUBLIC KEY PEM file instead of the string that you can attach into their ~/.ssh/authorized_keys file, you can get it with `ssh-keygen -i -f file.pem`.

## Importing keypair into AWS

```
$ aws ec2 import-key-pair --key-name juha.ylitalo@gmail.com \
> --public-key-material file://.ssh/id_rsa.pub
{
    "KeyName": "juha.ylitalo@gmail.com",
    "KeyFingerprint": "..."
}
```

Example works on RSA key, but if you try same thing with DSA keys, you will get error message claiming that "Key is not in valid OpenSSH public key format".
