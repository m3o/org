# V3 Cleanup

There is some legacy code copied over from go-micro which no longer makes sense in Micro. This document outlines a couple of examples with proposals for how to solve them.

## Options

In go-micro, the individual packages would be configured with the implementations used of other interfaces. For example, the router package has a Registry option. Now we have the public functions in Micro, the router can now call use these instead of maintaining it's own reference to the registry.

In summary, where a package references another package in the format `s.options.Registry.ListServices()`, this would be updated to simply be `registry.ListServices`.

## Client.Publish

The client interface still has a Publish function. This function differs slightly in it's implementation to broker since it accepts an interface as an argument vs []byte, making it easier to use. I propose updating the broker interface to look like the following, and then removing Publish/Subscribe from server and client packages respectively.

```go
// Broker is an interface used for asynchronous messaging.
type Broker interface {
	Publish(topic string, body interface{}, opts ...PublishOption) error
	Subscribe(topic string, opts ...SubscribeOption) (<-chan Message, error)
}

// Message is the object returned by the broker when you subscribe to a topic
type Message struct {
	// ID to uniquely identify the message
	ID string
	// Topic of message, e.g. "registry.service.created"
	Topic string
	// Timestamp of the message
	Timestamp time.Time
	// Metadata contains the values the message was indexed by
	Metadata map[string]string
	// Body contains the encoded message
	Body []byte
}

// Unmarshal the message body into an object
func (m *Message) Unmarshal(v interface{}) error {
	return json.Unmarshal(m.Body, v)
}
```