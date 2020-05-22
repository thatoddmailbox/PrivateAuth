# PrivateAuth
PrivateAuth is an extension to the [IndieAuth](https://indieauth.spec.indieweb.org/) protocol. It's intended to allow authentication to small applications with a limited audience; for example, a private media server, a home automation system, or a personal blog.

It tries to be as compatible as possible with IndieAuth from an endpoint perspective. A PrivateAuth endpoint should be able to handle logins for both PrivateAuth and IndieAuth clients. That way, a user may reuse an existing IndieAuth endpoint with a PrivateAuth client, or vice-versa.

To a certain degree, this compatibility extends to clients; however, clients that support both IndieAuth and PrivateAuth logins _MUST_ verify the domain of PrivateAuth logins. In other words, they _MAY NOT_ skip the domain check as part of the "Authentication code verification" section below.

## Definitions
A _restricted client_ is one that wishes to limit the users that can sign into it. A _full client_ is one that allows any user to sign in.

A user's _profile data_ is comprised of the additional fields (`name`, `shortName`, and `username`) specified in the "Authentication code verification" section below.

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

When making the verification request, the client _MUST_ add the X-PrivateAuth-Version header, with a value of 1. Endpoints that receive valid requests with this header _MUST_ have a JSON response containing fields according to the following table:
| Field | Status | Description | Example |
| ----- | ------ | ----------- | ------- |
| `me` | **required** | This has the same meaning as it does in the IndieAuth specification. | `"https://alex.studer.dev"`
| `name` | optional | The user's full name. | `"Alex Studer"` |
| `shortName` | optional | A short version of the user's full name. | `"Alex"` |
| `username` | **required** | A _unique_ identifier of the user's account at the endpoint. | `"alex"` |
| `permissions` | **required** | An array of a user's permissions. See the "Permissions" section below. | `["sampleapp:read", "differentapp:documents.read"]` |

The `username` _MUST_ be unique between different users on the same endpoint, but _MAY_ be the same between different users on different endpoints. The `name` and `shortName` fields do not have any uniqueness requirements.

For compatibility reasons, clients _MUST_ be able to ignore additional fields beyond what is specified here. That is, if a future revision of the specification adds additional fields, existing clients _MUST_ ignore the additions.

## Permissions
PrivateAuth adds the concept of "permissions": arbitrary strings associated with a user that can be interpreted by clients. Permissions _SHOULD_ be always in lowercase, and _MUST_ be verified in a case-insensitive manner.

### Conventions
_This section is non-normative._

While the specification does not strictly impose requirements on what permission strings look like, the convention is to user the client's name, a colon (`:`), and a descriptive string of the permission. If some hierarchy is needed in the descriptive string, a period (`.`) may be used.

For example, a client named "SampleApp" might look for the permissions `sampleapp:read`, `sampleapp:write`, and `sampleapp:admin`. A cloud storage client named "DifferentApp" might have permissions `differentapp:documents.read`, `differentapp:documents.write`, `differentapp:photos.read`, and `differentapp:photos.write`.

A client _SHOULD_ report an error if it finds a permission with its application name that it does not recognize, to protect against configuration errors.

## Roles
_This section is non-normative._

The PrivateAuth specification does not have a concept of a user's roles. However, endpoints _MAY_ implement this concept by allowing an administrator to define roles as lists of permissions, and then allow assigning users to those roles.

For example, an administrator could configure their endpoint to have an "Editor" role with permissions of `["sampleapp:read", "sampleapp:write", "differentapp:documents.read", "differentapp:documents.write"]`. Then, any users with this role assigned would automatically have those permissions added to their overall `permissions` array.

## Implementations
* [OpenResty script](https://github.com/thatoddmailbox/privateauth-openresty)
* [unifi-blink](https://github.com/thatoddmailbox/unifi-blink)