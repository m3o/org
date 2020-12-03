# Micro CLI Commands:

## micro login

### Currently

`micro login` currently calls Auth.Generate, using the current env vars as configuration, e.g. if 
the profile set is "server", it'll perform an RPC to go.micro.auth Auth.Generate. This will 
generate an auth account scoped to this environment.


### Future

Each user will have one account which is known as their Micro account. Irrespective of what 
namespace the user is running in, their credentials will always be the same. `micro login` will
now ask the user for their email, and then perform an RPC via proxy.m3o.com:8081 (the default
proxy), to an Account service in the micro/services repo. e.g. go.micro.service.account 
Account.RequestLogin. 

The service will send an email to the user with a one time password, e.g. "4923-3983-2422" which 
they copy to the CLI and is then sent on a second RPC to Account.VerifyLogin. If the token matches
and an account with this email already exists, the user will be logged in (creds will be generated
for them and the CLI will cache these).

If a user account doesn't exist, we'll ask the user for their first and last name, then create an 
account using this infomation (Account.Create) and the email token to verify.


## micro namespace list

### Currently

This command does not yet exist

### Future

This command will list all the namepaces the user has access to by performing an RPC to the 
namespace service in the micro/services repo, e.g. go.micro.service.namespaces Namespaces.List. 
They will be listed in addition to "local", which is always present. e.g.:

```
* local
  kytra-production
  kytra-staging
```

## micro namespace create

### Currently

This command does not yet exist

### Future

If the user is not logged in, the user will be prompted to call `micro login` before they can 
use this command. 

If the user does not provide the name as the first argument, the user will be asked to provide a name
 for the project. The name will be validated to be unique and an error will be returned if it's not.

The CLI will execute an RPC to the Namespace Service (go.micro.service.namespace) which will firstly
 validate the user has a payment method setup. If the user doesn't, a short-lived access token will 
 be generated and used to generate the following link: https://pay.m3o.com/?tok=[token].

At pay.m3o.com, the token will be used to identify the user and then they can create a payment 
method via Stripe JS. The payment method will be created on the backend and then the user can retry 
the command.

If the user does have a payment method, a subscription will be created via the Payments Service 
(go.micro.service.payments.stripe - I think). The namespace and service accounts will then be 
created via the kubernetes service and the RPC will complete. 

## micro namespace delete

### Currently

This command does not yet exist


### Future

If the user is not logged in, the user will be prompted to call `micro login` before they can 
use this command. 

The CLI will ask for confirmation, then execute an RPC to the Namespace service 
(go.micro.service.namespaces Namespace.Delete), which will cancel the stripe subscription and delete 
the k8s resources (the inverse of the namespace create command).

## micro {call,logs,run,kill,update,...}

### Currently

These commands are using the go-micro interfaces as configured by the profiles. 

### Future

The commands will continue to use the go-micro interfaces as normal, however the profiles will either
be ServerCLI (for the local namespace) or the PlatformCLI (for any other namespace, e.g. 
"kytra-production").

Before each call, if there are no credentials cached for the namespace, the micro internal client 
will execute an RPC to get a short-lived JWT to execute the request. The RPC will call a service 
such as go.micro.service.namespace Namespace.Auth, which will generate a set of auth credentials 
which has the issuer of the namespace. These credentials will be set in the Authorization header of 
each call. 

The credentials returned will be an access token and a refresh token, whilst the access token is 
valid, it can be used to make calls to the platform. Once it expires, Auth.Refresh should be called 
to exchange the credentials for a fresh set. 