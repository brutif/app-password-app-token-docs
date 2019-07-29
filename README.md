
Here's draft documentation for the app-password and app-token functionality being added to Liberty's OpenId Connect provider.

The general flow of things might be -

1) What they are and how to choose (below)
2) Links to detailed configuration instructions (links below)

3) Documentation of the new ui URL's (tbd)

#### 1: About Application passwords and application tokens
### Using application passwords and application tokens

Non-browser applications can use long-lived application passwords or application tokens to authenticate users, so the users do not
have to supply their password to the non-browser application.  Users first authenticate to the OpenID Connect provider using a REST interface and obtain a long-lived application password or application token.  Next, the token or password is supplied to the non-browser application. The application can use these to authenticate on the user's behalf and access resources that are protected by OpenID Connect.   The user's password is never exposed to the non-browser application. Application passwords and application tokens can be revoked if necessary without changing the user's password.

### Choosing application passwords or application tokens

Application tokens are long-lived tokens that are submitted to the protected resource.  If they are compromised, they can be used for an extended period of time until they expire or are revoked.

Application passwords are not used directly but are exchanged repetitvely for short-lived tokens that are submitted to the protected resource.  If they are compromised, they won't be valid for long.  However the use of application passwords requires the non-browser application to manage the recurring exchange, and also for the provider to support the ROPC (Resource Owner Password Credential) grant type to perform the exchange.  Using application passwords should be considered where these additional conditions can be met. 


[app-passwords](./app-passwords.md)
[app-tokens](./app-tokens.md)


