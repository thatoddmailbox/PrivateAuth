# PrivateAuth
PrivateAuth is an extension to the [IndieAuth](https://indieauth.spec.indieweb.org/) protocol. It's intended to allow authentication to small applications with a limited audience; for example, a private media server, a home automation system, or a personal blog.

It tries to be as compatible as possible with IndieAuth from an endpoint perspective. A PrivateAuth endpoint should be able to handle logins for both PrivateAuth and IndieAuth clients. That way, a user may reuse an existing IndieAuth endpoint with a PrivateAuth client, or vice-versa.

To a certain degree, this compatibility extends to clients; however, clients that support both IndieAuth and PrivateAuth logins _MUST_ verify the domain of PrivateAuth logins. In other words, they _MAY NOT_ skip the domain check as part of the "Authentication code verification" section below.

## Definitions
A _restricted client_ is one that wishes to limit the users that can sign into it. A _full client_ is one that allows any user to sign in.

## Authentication
The authentication flow is very similar to IndieAuth. You might want to compare with [section 5 of the IndieAuth spec](https://indieauth.spec.indieweb.org/#authentication).

### Discovery
A restricted client _MAY_ skip the authorization endpoint discovery step. In this case, the restricted client _SHOULD_ provide a configuration mechanism to allow an administrator to set a trusted authorization endpoint.

Clients _MAY_ also provide a configuration mechanism to determine if they should behave in either a restricted or full manner.

### Authentication request
When building the authentication request, a restricted client _MAY_ omit the `me` parameter, and rely on the authorization endpoint to determine the user's profile URL. This breaks compatibility with IndieAuth endpoints, and so should be only done if a developer is certain that their restricted client will only be used with PrivateAuth endpoints.

If an authorization endpoint receives a request without the `me` parameter, it _MUST_ somehow resolve the user's profile URL. How exactly this happens is dependent on the specific implementation&mdash;for example, a user could be prompted to enter their profile URL, or the endpoint could automatically calculate a profile URL based on a username entered by the user.

## Authentication response
There are no changes to this section from the IndieAuth specification.

## Authentication code verification
If a restricted client chose to omit the `me` parameter in the intial request, then it is not possible to verify that the returned profile URL's domain is correct. Therefore, this domain check _MAY_ be skipped by restricted clients.

> **Important:** this domain check _MAY NOT_ be skipped in a client that supports both IndieAuth and PrivateAuth logins, as this would be a security flaw. Both IndieAuth clients and PrivateAuth full clients must be able to reject logins from a malicious endpoint&mdash;that is, an endpoint that claims to authenticate users for a domain that it does not control.

## Implementations
* [OpenResty script](https://github.com/thatoddmailbox/privateauth-openresty)
* [unifi-blink](https://github.com/thatoddmailbox/unifi-blink)