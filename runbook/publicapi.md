# Public API

Details for the public api service

## V1 Serving

All requests for the v1 api are served through `/v1` via the `v1api` service.

## The admin token

We need an admin token for github.com/micro/services to publish the publicapis. 

```
micro call auth Auth.Token '{"id":"asim","secret":"YOURPASSWORD", "token_expiry":1922557662}' 
```

Set the `MICRO_ADMIN_TOKEN` value
