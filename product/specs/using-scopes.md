# Using scopes - different uses on the platform

## Goal

Exposing the APIs to consumers, authorizing them when doing writes (or any action really).

## Use case

We don't want to create physical namespaces for each user signing up to Backend.
We would like to get have them as proper Micro accounts however.

## Problem

If a user has an account, they can do store read, config read, run services etc. which is something we don't want to enable for people just calling existing services inside Backend.

## Solution

Make each existing user have the scope "admin", and change existing endpoints to check for this scope.
Users of Backend would not have this scope so they could do little apart from calling services.