# API consumption



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


## Outstanding questions
Should we support a Sandbox env (similar to Stripe) for testing APIs before moving on to the paid stuff? 

GraphQL?
