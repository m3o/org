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

## From Services

It is simiarly easy to access and set config values from a service.
A good example of reading values is [the config example test service](https://github.com/micro/micro/tree/master/test/service/config-example):

```go
package main

import (
	"fmt"
	"time"

	"github.com/micro/micro/v3/service"
	"github.com/micro/micro/v3/service/config"
)

type keyConfig struct {
	Subkey  string `json:"subkey"`
	Subkey1 int    `json:"subkey1"`
}

type conf struct {
	Key keyConfig `json:"key"`
}

func main() {
	go func() {
		for {
			time.Sleep(time.Second)
			val, err := config.Get("key.subkey")
			fmt.Println("Value of key.subkey: ", val.String(""), err)

			val, err = config.Get("key", config.Secret(true))
			if err != nil {
				fmt.Println(err)
			}
			c := conf{}
			err = val.Scan(&c.Key)
			fmt.Println("Value of key.subkey1: ", c.Key.Subkey1, err)
		}
	}()

	// run the service
	service.Run()
}
```

The above service will print the value of `key.subkey` and `key.subkey` every second.
By passing in the `config.Secret(true)` option, we tell config to decrypt secret values for us, similarly to the `--secret` CLI flag.

The [config interface](https://github.com/micro/go-micro/blob/master/config/config.go) specifies not just `Get` `Set` and `Delete` to access values,
but a few convenience functions too in the `Value` interface.

It is worth noting that `String` `Int` etc methods will do a best effort try at coercing types, ie. if the value saved is a string, `Int` will try to parse it.
However, the same does not apply to the `Scan` method, which uses `json.Unmarshal` under the hood, which we all know fails when encountering type mismatches.

`Get` should, in all cases, return a non nil `Value`, so even if the `Get` errors, `Value.Int()` and other operations should never panic.