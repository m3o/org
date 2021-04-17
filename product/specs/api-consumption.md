# API consumption

Thoughts on what is needed to refocus to an API consumption model.

## Scenarios

### Inbound to m3o
How does a user find themselves on m3o.com and using our APIs. We need to support the core flow of `google for "API for X" -> land on m3o.com docs for the relevant API -> sign up -> integrate`

1. SEO friendly content (blog pages and crawlable API docs). Everything public, nothing behind login. Login should add functionality like API key or similar but everything should be viewable without.
1. API docs homepage should have search/exploration etc (see below)
1. Docs and blog should have login/signup CTA if user not logged in


#### Pages
##### API docs home page
There should be an exploration mechanism. Likely a list of available APIs and a way to search through them
- search bar. Search needs to be more than searching on name e.g. `"address lookup"` should return the geocoding API.
- When we have more APIs such that the list gets unwieldy we might want to do carousel/highlighted APIs/something else - think app store layout.
- Each API has a summary text with click through to a details page

##### API Details page
- Summary description
- Docs (interactive?)
- Examples
- Pricing

### Basic API Consumer flow
The core API consumer flow. We need to support the core flow of `sign up -> find an API to use -> integrate it with my code -> pay for usage`   

1. User lands on m3o.com. Main experience/content based around API consumption. Large CTA to login/register
1. User registers (either username/password or github oauth) or logs in
1. Lands on their home page. Could be the API docs home page, or perhaps a summary with link to API docs home
1. Explores/discovers using the API docs home page. Integrates code
1. Creates API key on usage page
1. Tops up their account on subscription page

#### Pages
##### m3o.com
- Rejigged to emphasise API consumption
- Highlights Distributed as a tech showcase
- Link "want to build APIs?"

##### API usage page
Usage page to show single pane view of their current usage
   - Show quota usages for all APIs (graphs etc)
   - Create new API key, see existing ones
   - CTA to upgrade, topup account

##### Subscription / payment page
- save/edit payment details
- see current balance
- top up account

    
## Scenario: API Builder 
API Builder flow. Need to support the core flow of `sign up -> build some micro based API -> hit it from external`

1. User lands on m3o.com. Link "want to build APIs?"
1. Builder page with blurb including
   - Sign up for platform
   - This is a beta so no SLAs etc
   - keep it free. means it stays simple to operate for now and we also prime the pump of available APIs (two sided market place problem). Eventually we'll move to paid tiers with scaling etc but not needed now 
1. no API key integration etc, can just offer it as open to the world under their namespaced custom url https://foo-bar-baz.m3o.com/service/call


### Phase 2 (6-12 months away)
Integration similar to our own so builders can offer APIs for other users to use
   - Ability to generate beautiful docs similar to our own 
   - Ability to charge / earn money
   - URL for use would probably change to https://api.m3o.com/v1/service/call so we can use v1api and quota service to do the accounting but need to work out how to route that back to the builder's namespace.


## Pricing
Pricing is complex and could be done in a multitude of different ways. We can start off with a simple model and adapt as we go along. For any pricing model, the fundamental data point we need to capture is the number of requests made to a particular API. We do this currently in the quota service which ticks up a redis counter. 

### Pay as you go
The pay as you go pricing model keeps it simple by asking users to top up their balance which gets decremented with usage. We will maintain a single balance for a user which is then used against all API usage e.g. using the geocoding API and the users API will decrement the same pot of money.

For now, to keep it simple, every API will have a flat rate i.e. no tiered pricing just Xp/request where X could be 0 (unlikely case where someone wants to offer a free API) or even fractional (where you want to charge 1p per 100 requests or similar).

### Free trial
If we want to enable a "free trial" model the easiest thing would be to give each user free credit, say £3, which would be enough for a few hundred requests.

## Accounting 
Count API usage to correctly debit account. 
When balance hits zero their service should stop.

User adds money in stripe -> inbound webhook -> update 
User uses API -> balance service counts -> if counter hits zero, block at the v1api 
Use redis counter to do distributed counting and maintain a running balance

if redis fails it has a backup

## Users
API consumers signup and are given an account in the `micro` namespace with type `customer` and role `customer`. 

API builders signup and are given an account in the `micro` namespace with type `developer` and role `developer`. They will be given a namespace with a separate admin account (similar to what they have in the current model). Scope to extend and give builders multiple namespaces (AKA projects). Remove signup from CLI.

## API publishing
An API is defined by a combination of 
- OpenAPI spec (JSON)
- Description
- Pricing

To "publish" an API we expect at least the above pieces of information. This is different to the backend service definition approach we currently take with protobufs. This is intentional; considering that Swagger/OpenAPI is fast becoming the industry standard for API specs it makes sense to adopt it. The tooling for open API UIs etc is also much better than hand crafting something. We will continue to use protobufs to define backend interprocess communication, indeed you can even generate protos from the openapi spec if you need. 

## Outstanding questions
Should we support a Sandbox env (similar to Stripe) for testing APIs before moving on to the paid stuff? 

GraphQL?
