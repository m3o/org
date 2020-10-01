---
layout: default
title: Private Repos
nav_order: 1
permalink: /getting-started/private-repos
parent: Get Started
---

# Using Private Repos

M3O supports private repos by passing your git credentials securely to our runtime as secrets. To enable private 
repos simply store your credentials locally with the following command.

```
micro user config set git.credentials.[org]
```

## GitHub

[This page](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) explains how to get a personal access token from GitHub.

Issue this command locally to save your personal access token to your local Micro config:
```
micro user config set git.credentials.github $your-personal-access-token
```

## GitLab

[This page](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) explains how to get a personal access token from GitLab.

Issue this command locally to save your personal access token to your local Micro config:
```
micro user config set git.credentials.gitlab $gitlab-username:$your-personal-access-token
```

Please note the `username` requirement above.

## Bitbucket

[This page](https://confluence.atlassian.com/bitbucketserver/personal-access-tokens-939515499.html) explains how to get a personal access token from Bitbucket.

Issue this command locally to save your personal access token to your local Micro config:
```
micro user config set git.credentials.bitbucket $bitbucket-username:$your-personal-access-token
```

Please note the `username` requirement above.

## Git repos hosted on custom domains

Many people have their own Gitlab etc. installations.
To set up credentials in cases like that, do the following:

```
micro user config set git.credentials.url domain.com
micro user config set git.credentials.token $username:$your-personal-access-token
```
