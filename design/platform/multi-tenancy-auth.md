# Multi-Tenancy Auth

Each module in go-micro (registry, router etc) has a domain specific name for "namespace" and every
implementation should support this argument. For example, the registry has a "domain" option which 
can be passed on each call, e.g. `registry.Register(srv, registry.RegisterDomain("foo"))`. Every 
implementation of the registry supports these paramaters and if the implementations are accessed
directly, anyone can query using the any domains etc.

Micro provides an abstractions via RPC to these implementations. For example, rather than using the 
etcd registry implementation directly, a go-micro service would use the service registry which would
be responsible for authenticating an authorizing every request.

micro has a package "internal/namespace" which provdies an Authorize method. This method takes a 
context and namespace and returns an error if the context (or more specificall - the auth account 
embedded in the context) cannot access the given namespace. At present, anyone can access the 
micro namespace in any model. For any non-micro namespace, the function will check to see if the
callers account was issued by the namespace it's requesting. The function and an example useage are
documented below:


```go
// Authorize will return an error if the context cannot access the given namespace
func Authorize(ctx context.Context, namespace string) error {
	// anyone can access the default namespace
	if namespace == DefaultNamespace {
		return nil
	}

	// accounts are always required so we can identify the caller. If auth is not configured, the noop
	// auth implementation will return a blank account with the default namespace set, allowing the caller
	// access to all resources
	acc, ok := auth.AccountFromContext(ctx)
	if !ok {
		return ErrUnauthorized
	}

	// the server can access all namespaces
	if acc.Issuer == DefaultNamespace {
		return nil
	}

	// ensure the account is requesing access to it's own namespace
	if acc.Issuer != namespace {
		return ErrForbidden
	}

	return nil
}
```

```go
// Register a service
func (r *Registry) Register(ctx context.Context, req *pb.Service, rsp *pb.EmptyResponse) error {
	var opts []registry.RegisterOption
	var domain string

	// parse the options
	if req.Options != nil && len(req.Options.Domain) > 0 {
		domain = req.Options.Domain
	} else {
		domain = registry.DefaultDomain
	}
	opts = append(opts, registry.RegisterDomain(domain))

	// authorize the request
	if err := namespace.Authorize(ctx, domain); err == namespace.ErrForbidden {
		return errors.Forbidden("go.micro.registry", err.Error())
	} else if err == namespace.ErrUnauthorized {
		return errors.Unauthorized("go.micro.registry", err.Error())
	} else if err != nil {
		return errors.InternalServerError("go.micro.registry", err.Error())
	}

	// register the service
	if err := r.Registry.Register(service.ToService(req), opts...); err != nil {
		return errors.InternalServerError("go.micro.registry", err.Error())
	}

	// publish the event
	go r.publishEvent("create", req)

	return nil

```

Hence, as long as all requests go via the micro server, access to each module is authorized. To 
start a service in a given namespace, the "MICRO_NAMESPACE" env var can be set, the service will 
then use this namespace as the Namespace option when registering the service, loading config etc. 

When a RPC is executed via a go-micro service, it's wrapped by the AuthClient wrapper (util/wrapper)
which injects the services auth credentials in the Authorization header. The micro serivce (e.g. the
registry decodes this header and injects the resulting auth account into the requests context, for
use by functions such as namespace.Authorize)

There is still some work left on this, the config implementation should be merged today and also the
auth implementation. We only recently added Namespace / Domain / Network options to the go-micro 
interfaces so up until that point, we were using the Micro-Namespace header to determine the namespace
the caller was requesting, which was suboptimal. Hence, there has been some work to update each micro
service to support the domain specific namespace option and utilize the micro namespace package to 
authorize each request.

Two things which are missing are:
• Verifying services on Auth.Generate: because we don't yet have identity / certs working, anyone
can generate auth accounts meaning the whole system is unlocked.
• Identifying server: the server (such as the proxy, router etc) needs access to all namespaces,
currently that's done by checking for "micro" as the auth account's issuer, however it might make
sense to change this to something else since "micro" is the default.
