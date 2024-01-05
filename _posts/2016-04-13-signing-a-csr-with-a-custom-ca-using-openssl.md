---
layout: post
current: post
navigation: True
title: Signing a CSR with a custom CA using openSSL
author: l4sh
date: 2016-04-13
category: tech
class: post-template
subclass: post
tags:
  - sysadmin
comments: true
---

Sometimes we need to self-sign an SSL certificate and of course the internet is
filled with guides on how to do it. But what about when we need to sign an
already issued Certificate Signing Request with our own Certificate Authority?

Most of what I’ve found on this topic requires a few steps to get this done and
I must say that one of the most detailed guides I have found is at
https://jamielinux.com/docs/openssl-certificate-authority/index-full.html.
However this post does not intend to be a fully detailed guide, but a quick
and dirty way to to create a CA and sign a CSR without much hassle.

Be aware that if you’re using these for a public website the certificate
generated through this method will prompt an error on browsers since we’re
not a recognized CA. If you’d like a free valid certificate you could use
Let’s Encrypt or StartSSL.

If you’re going to be issuing a lot of certificates it’s best if you assemble
the structure explained in Jamie Nguyen’s article in order to have more control.

The process
The first thing to do is create a work directory and cd into it.

```terminal
mkdir custom_CA
cd $_
```

Then we generate a new CA key file inside this directory.

```terminal
openssl genrsa -out rootCA.key 2048
```

Now we need a certificate to sign the request. This will ask some information such as country, state, hostname, etc. Keep in mind that this is the CA’s information and not the site’s or entity that requests the certificate information.

```terminal
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
```

Then we’ll copy the CSR file into the folder we’re working on. In this case the file is named example_com.csr and currently resides in my home folder.

```terminal
cp ~/example_com.csr custom_CA/
```

Sign the CSR generating a new certificate.

```terminal
openssl x509 -req -in example_com.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out example_com.crt -days 365 -sha256
```

As a final step we verify that the generated certificate matches the signing request. For this we can use the modulus option in openssl and generate an MD5 hash and visually compare. The output of both commands should be identical.

```terminal
openssl req -noout -modulus -in example_com.csr | openssl md5
openssl x509 -noout -modulus -in example_com.crt | openssl md5
```
