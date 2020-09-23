---
layout: page
title: Authentication
keywords: micro
tags: [micro]
permalink: /tutorials/authentication
summary: Authentication and authorization
parent: Tutorials
nav_order: 2
toc_list: true
---

# Authentication and authorization
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

Micro (and consequently M3O) comes with fine tunable authentication and authorization capabilities.
This tutorial assumes you are already logged in. For M3O that means you have signed up, for a local `micro server` it means doing a `micro login` and logging in with
id `admin` and password `micro`:

```
$ micro login
Enter email address: admin
Enter Password: 
Successfully logged in.
```

## Rules

Rules determine what resource a user can access. The default rule is the following:

```sh
$ micro auth list rules
ID          Scope           Access      Resource        Priority
default     <public>        GRANTED     *:*:*           0
```

The `default` rule makes all services callable that appear in the `micro status` output.
Let's see an example of this.

```sh
$ micro run helloworld
$ namespace=$(mico user namespace)
$ curl -H "Micro-Namespace: $namespace" https://api.m3o.com/helloworld
# Locally:
# curl 127.0.0.1:8080/helloworld/call?name=Alice
{"msg":"Hello Alice"}
```

If we want to prevent other people from calling our services, we can create the following rule

```sh
# This command creates a rule that enables only logged in users to call the micro server
micro auth create rule  --access=granted --scope='*' --resource="*:*:*" onlyloggedin
```

and delete the default one.
Here, the scope `*` is markedly different from the `<public>` scope we have seen earlier when doing a `micro auth list rules`:

```sh
$ micro auth list rules
ID			    Scope			Access		Resource		Priority
onlyloggedin	*			    GRANTED		*:*:*			0
default			<public>		GRANTED		*:*:*			0
```

Now, let's remove the default rule.

```sh
# This command deletes the 'default' rule - the rule which enabled anyone to call the 'micro server'.
$ micro auth delete rule default
Rule deleted
```

Let's try curling our service again:

```sh
$ micro run helloworld
$ namespace=$(mico user namespace)
$ curl -H "Micro-Namespace: $namespace" https://api.m3o.com/helloworld
# Locally:
# curl 127.0.0.1:8080/helloworld/call?name=Alice
{"Id":"helloworld","Code":401,"Detail":"Unauthorized call made to helloworld:Helloworld.Call","Status":"Unauthorized"}
```

Great success! This means our `onlyloggedin` rule took effect. We can still call the service with a token:

```sh
$ micro run helloworld
$ token=$(micro user token)
$ curl -H "Micro-Namespace: $namespace" "Authorization: Bearer $token"  https://api.m3o.com/helloworld
# Locally:
# curl "Authorization: Bearer $token" 127.0.0.1:8080/helloworld/call?name=Alice
{"msg":"Hello Alice"}
```

(Please note tokens have a limited lifetime so the line `$ token=$(micro user token)` has to be reissued from time to time, or the command must be used inline.)


These are the very basics of auth. Please check back from time to time for more!