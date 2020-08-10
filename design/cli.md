# CLI

## Dynamic CLI Commands

Micro supports dynamic CLI commands, allowing users to call their services without needing to manually add commands to Micro. For example:

• `micro helloworld` will call helloworld Helloworld.Call. If one argument is provided, it's assumed this is the name of the service, and we want to execute the Call RPC.
• `micro helloworld foo` will call helloworld Helloworld.Foo. If a second argument is provided, we use this as the method name, and keep the service name as the handler. 
• `micro helloworld debug health` will call helloworld Debug.Health. The second and third arguments are combined to create the RPC endpoint.

The dynamic CLI commands use the registry to determine which flags to support. Top level arguments can be passed using their name as the flag, e.g. `micro helloworld name=Ben`. The CLI  will map nested arguments using '_'. For example, the following request can be called as `micro helloworld --options_name=Ben`:

```go
type HelloworldRequest struct {
  Options struct {
    Name string
  }
}
```


If an unknown flag is passed, an error will be returned, e.g. `Unknown flag: foo`.

The CLI will output the response in JSON format, e.g

```bash
> micro helloworld --name=John      
{
	"msg": "Hello John"
}
```