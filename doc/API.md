# Foodsoft API

Foodsoft provides a JSON REST API that gives access to operations like
like listing open orders, updating the ordergroup's order, and listing financial
transactions. Not all Foodsoft functionality is available through the API, but
we're open for new additions.

The API is documented using [Open API 2.0](https://github.com/OAI/OpenAPI-Specification)
/ [Swagger](https://swagger.io/) in [swagger.v1.yml](swagger.v1.yml).
This provides a machine-readable reference that is used to provide documentation.

**Note:** the current OAuth scopes may be subject to change, until the next release of Foodsoft.

## API endpoint documentation

&gt;&gt; [View API documentation](http://petstore.swagger.io/?url=https%3A%2F%2Fraw.githubusercontent.com%2Ffoodcoops%2Ffoodsoft%2Fmaster%2Fdoc%2Fswagger.v1.yml) &lt;&lt;

The above documentation can communicate with the API directly on a local development
installation of Foodsoft at [http://localhost:3000/f](http://localhost:3000/f).

You'll need to give access to the application first. This can be done by going to
_Administration_ > _Configuration_ > _Apps_ in Foodsoft. Select _New Application_,
enter any name, put `http://petstore.swagger.io/oauth2-redirect.html` in _Redirect URI_
and disable _Confidential_. After submission, you will have an _Application UID_ that
you can enter that as `client_id` after clicking _Authorize_ in the Swagger UI.


## Security

Uses the [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) gem,
which provides an OAuth2 provider.


### Authorization code flow

This is the recommended flow for server-side web applications, where
members login with Foodsoft, then redirected to the app, which then obtains
an access token using the authorization code supplied at redirection.

Before you can obtain an access token, the client needs to obtain an id and secret.
(You can currently skip this for the password credentials flow.) This needs to be
done for each Foodsoft scope by an admin.

1. Click on the _Apps_ button at the right in Foodsoft's configuration screen.
2. Click on _New application_
3. Enter any _Name_ and put the website of your app in _Redirect URI_ and _Submit_.
4. Click on the new applications' name for the app id and secret.
5. To quickly test, logging into the app, press _Authorize_.

Note that the user doesn't need to confirm that he is giving the app access to his
Foodsoft account by default, since apps can only be created by admins. If you
want to change that, see disable `skip_authorization` in `config/initializers/doorkeeper.rb`.

[Read more](https://github.com/doorkeeper-gem/doorkeeper/wiki/Authorization-Code-Flow).


### Implicit flow

This is the recommended flow for client-side web applications. It looks a lot
like the authorization code flow, but when redirecting back to the app, the
access token is available directly as part of the url _fragment_ (`window.location.hash`).

This flow also needs to be registered in Foodsoft as in the authorization code flow,
but with _Confidential_ disabled. You only need the `client_id`, not the secret.

**note** please make sure you understand sections
[4.4.2](http://tools.ietf.org/html/rfc6819#section-4.4.2) and
[4.4.3](http://tools.ietf.org/html/rfc6819#section-4.4.3) of the OAuth2 Threat
Model document before using this flow.

You may find Doorkeeper's [implicit_grant_test](https://github.com/doorkeeper-gem/doorkeeper/blob/master/spec/requests/flows/implicit_grant_spec.rb) useful.


### Password credentials flow

To obtain a token using a username/password directly, you can do this:

```ruby
require 'oauth2'
c = OAuth2::Client.new('client_id', 'secret', site: 'http://localhost:3002/f/', authorize_url: 'oauth/authorize', token_url: 'oauth/token')
c.password.get_token('admin', 'secret').token
# => "1234567890abcdef1234567890abcdef1234567890abcdef123456790abcdef1"
```

Now use this token as value for the `access_token` when accessing the API, like
http://localhost:3002/f/api/v1/financial_transactions/1?access_token=12345...

[Read more](https://github.com/doorkeeper-gem/doorkeeper/wiki/Client-Credentials-flow).


## Logout

When the user logs out of Foodsoft, all access tokens are destroyed, except when
the token's scope includes `offline_access` (so offline applications are possible).