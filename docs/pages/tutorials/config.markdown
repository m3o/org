---
layout: page
title: Config
keywords: micro
tags: [micro]
permalink: /tutorials/config
summary: Configuration
parent: Tutorials
nav_order: 4
toc_list: true
---

# Config
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

Config is a way to store configuration data for services. Config is both accessible from the Micro CLI and from services themselves.

## CLI usage

Let's assume we have a service called `helloworld` from which we want to read configuration data.
First we have to insert said data with the cli. Config data can be organized under different "paths" with the dot notation.
It's a good convention to save all config data belonging to a service under a top level path segment matching the service name:

```sh
$ micro config set helloworld.somekey hello
$ micro config get helloworld.somekey
hello
```

We can save an other key too and read all values in one go with the dot notation:

```sh
$ micro config set helloworld.someotherkey "Hi there!"
$ micro config get helloworld
{"somekey":"hello","someotherkey":"Hi there!"}
```

As it can be seen, the config (by default) stores configuration data as JSONs.
We can save any type:

```sh
$ micro config set helloworld.someboolkey true
$ micro config get helloworld.someboolkey
true
$ micro config get helloworld
{"someboolkey":true,"somekey":"hello","someotherkey":"Hi there!"}
```

So far we have only saved top level keys. Let's explore the advantages of the dot notation.

```sh
$ micro config set helloworld.keywithsubs.subkey1 "So easy!"
{"keywithsubs":{"subkey1":"So easy!"},"someboolkey":true,"somekey":"hello","someotherkey":"Hi there!"}
```

Some of the example keys are getting in our way, let's learn how to delete:

```sh
$ micro config del helloworld.someotherkey
$ micro config get helloworld
{"keywithsubs":{"subkey1":"So easy!"},"someboolkey":true,"somekey":"hello"}
```

We can of course delete not just `leaf` level keys, but top level ones too:

```sh
$ micro config del helloworld.keywithsubs
$ micro config get helloworld
{"someboolkey":true,"somekey":"hello"}
```

### Secrets

The config also supports secrets - values encrypted at rest. This helps in case of leaks, be it a security one or an accidental copypaste.
They are fairly easy to save:

```sh
$ micro config set --secret helloworld.hushkey "Very secret stuff" 
$ micro config get helloworld.hushkey
[secret]

$ micro config get --secret helloworld.hushkey
Very secret stuff

$ micro config get helloworld
{"hushkey":"[secret]","someboolkey":true,"somekey":"hello"}

$ micro config get --secret helloworld
{"hushkey":"Very secret stuff","someboolkey":true,"somekey":"hello"}
```

Even bool or number values can be saved as secrets, and they will appear as the string constant `[secret]` unless decrypted:

```sh
$ micro config set --secret helloworld.hush_number_key 42
$ micro config get helloworld
{"hush_number_key":"[secret]","hushkey":"[secret]","someboolkey":true,"somekey":"hello"}

# micro config get --secret helloworld
{"hush_number_key":42,"hushkey":"Very secret stuff","someboolkey":true,"somekey":"hello"}
```