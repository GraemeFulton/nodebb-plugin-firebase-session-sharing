# Session Sharing for NodeBB

In a nutshell, this plugin allows you to share sessions between your application and NodeBB. You'll need to set a
special cookie with a common domain, containing a JSON Web Token from Firebase Custom Auth with user data. 
If sufficient, this plugin will handle the rest (user registration/login).

## How is this related to SSO?

Single Sign-On allows a user to log into NodeBB through a third-party service. It is best (and most securely)
achieved via OAuth2 provider, although other alternatives exist. An example of a single sign-on plugin is
[nodebb-plugin-sso-facebook](https://github.com/julianlam/nodebb-plugin-sso-facebook).

Single sign-on *does not* allow a session to become automatically created if a login is made to another site.
This is the one misconception that people hold when thinking about SSO and session sharing.

This session sharing plugin will allow NodeBB to automatically log in users (and optionally, log out users)
if the requisite shared cookie is found (more on that below).

You can use this plugin and single sign-on plugins together, but they won't be seamlessly integrated.

## Compatibility

This plugin is compatible with NodeBB **v1.0.0 and up**. As of 27 January 2016, v1.0.0 has not been released
yet, so you will need to be running the `master` branch of a NodeBB installation via GitHub.

## How does this work?

This plugin checks incoming requests for a **shared cookie** that is saved by your Firebase application when a user
logs in. This cookie contains in its value, a specially crafted signed token containing unique identifying
information for that user.

If the user can be found in NodeBB, that user will be logged in. If not, then a user is created, and that
unique indentifier is saved for future reference.

## How can I integrate my site with this plugin?

When a user logs in, you'll need to save a cookie in their browser session for NodeBB. This cookie needs
to have a couple special attributes:

* The `HttpOnly` flag should be set for security, otherwise the shared cookie can be read by via AJAX/XHR
* The `domain` should be set to the naked domain. That is, if your site is at `app.example.com`, and your
forum is at `talk.example.com`, the cookie domain should be set to `example.com`.
* The cookie name is up to you, and can be configured in this plugin's settings. *(Default: `token`)*
* The cookie value is a [JSON Web Token](https://jwt.io/). See below.

### Generating the JSON Web Token

https://firebase.google.com/docs/auth/admin/create-custom-tokens

## Security

Please note that according to the JWT spec, the payload itself is ***not encrypted***, only *signed*. That is,
the Base64 Url Encoded payload is appended to the header. It can be decoded trivially (as base64 is not meant
to be cryptographically secure), so **do not put any private information in the payload**. The header and
payload themselves are *signed* against the secret, and NodeBB will only allow a JWT through if it has not been
tampered with. That is, NodeBB will only continue with a login if the signature can be independently generated
by the received payload and the secret.

Use secure cookies transmitted via HTTPS if at all possible.
