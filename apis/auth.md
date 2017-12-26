# auth service

Base URLs:

- prod: https://service-auth.prod.ext.cuvva.co
- sandbox: https://service-auth.sandbox.ext.cuvva.co

## API

Conforms to the [Cuvva services standard][1]. However, this service is *not*
RESTful. Instead, it follows more of a simplified RPC approach.

All methods are specified below. Methods are called as
`POST /2/:version/:method`.

The `:version` is replaced with the date you implemented the code using the API.
Versions must not be mixed and matched - all calls to this service must use the
same version string.

Neither the query string, nor any URL parameters are used. Input is only
supported via the body. JSON and urlencoded forms are supported for input.
Output is always JSON. Unless otherwise noted, methods do not require
authorization.

Where no response is specified for a method, no response will be provided. The
status code will be 204. The response may be changed to return data - with a
different 2xx status code - at any time. This would not be considered a breaking
change.

### `register_user`

Creates the user with unverified identifiers (other than the Facebook login) and
returns authentication data - including an access token, refresh token, etc.

If a user already exists for any of the identifiers, an `already_exists` error
will be thrown. If the format or contents of any identifier value is invalid,
`invalid_identifiers` will be thrown with various different inner errors
depending on the exact reason.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"email": "james@cuvva.com",
	"mobile_phone": "+447700900123",
	"facebook_token": "EAAWgC7Fr0sABABA3gzOviDuZDZD"
}
```

The `facebook_token` and `mobile_phone` fields are optional.

#### Response

```json
{
	"access_token": "eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkiLCJhdWQiOiIwNmM5MmYyYy04YmJiLTQwM2MtOWVlNy0zYjNiOGViMGIzMGYifQ.",
	"token_type": "bearer",
	"expires_in": 3600,
	"expires_at": "2016-06-01T00:00:00Z",
	"refresh_token": "02.571cbf2a5e72750100a41a5b.0335fc54f030a9a476d210854f4cb1f5def99f64ea063b806fde65563feb0c86",
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f"
}
```

### `authenticate`

Creates and returns authentication data - access token, refresh token, etc.

This conforms to the "Token Endpoint" defined in the
[OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.2) spec.

The `access_token` returned is short lived. The `refresh_token` allows you to
get a new access token once the original one has expired. Refresh tokens last a
long time, but can only be used once - they are replaced when used.

#### Request

Multiple methods of authenticating are available. Which one is determined by the
`grant_type`:

##### `identifier_token`

Authenticates based on a verified external identifier - email or Facebook login.

If the token is invalid, `invalid_token` will be thrown. If the token is valid
but no user exists for the identifier, `identifier_not_found` will be thrown.

**Note:** although mobile phone identifiers are currently supported for
authentication, this is now deprecated and will stop working in the near future.

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"grant_type": "identifier_token",
	"identifier_type": "email",
	"identifier_value": "james@cuvva.com",
	"identifier_token": "123456"
}
```

##### `refresh_token`

Authenticates with the same scope and entities as the original authentication
request, unless a lesser scope is requested. (Although at present, scopes are
not used or supported.)

If the refresh token provided does not exist, `invalid_token` will be thrown.
If this happens, the user will typically need to log in again from scratch.

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"grant_type": "refresh_token",
	"refresh_token": "02.571cbf2a5e72750100a41a5b.960b19318364fb3838b97695b27c36287fbd02d677131dfd41261f7d1bd3a62c"
}
```

#### Response

```json
{
	"access_token": "eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkiLCJhdWQiOiIwNmM5MmYyYy04YmJiLTQwM2MtOWVlNy0zYjNiOGViMGIzMGYifQ.",
	"token_type": "bearer",
	"expires_in": 3600,
	"expires_at": "2016-06-01T00:00:00Z",
	"refresh_token": "02.571cbbadf999da0100de3a70.0335fc54f030a9a476d210854f4cb1f5def99f64ea063b806fde65563feb0c86",
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f"
}
```

### `get_user`

Retrieves data for the user and returns it. Authorization is required.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

#### Response

```json
{
  "id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
  "email": "james@cuvva.com",
  "email_verified": true,
  "mobile_phone": "+447700900123",
  "mobile_phone_verified": false,
  "facebook_id": "1122521981125307"
}
```

### `send_identifier_token`

Generates and sends a verification token to the given identifier.

The token can then be used to authenticate with the identifier, or to replace
the identifier of the type on the account.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"type": "mobile_phone",
	"value": "+447700900123"
}
```

### `replace_identifier`

Verifies an identifier and replaces any existing one of the type on the account
for the given user. Authorization is required.

Eventually it may be possible to have multiple of the same type of identifier.
If that happens, the functionality of this method will most likely change to
replacing the primary identifier.

If the format or contents of an identifier value is invalid for the type,
`invalid_value` will be thrown with various different inner errors depending on
the exact reason. If the identifier is already used by another user, an
`already_exists` error will be thrown.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client_id": "285b13ee-e5a5-4391-990e-4b9c8da590f6",
	"type": "facebook",
	"value": "1122521981125307",
	"token": "EAAWgC7Fr0sABABA3gzOviDuZDZD"
}
```

### `detach_identifier`

Removes an identifier from the given user. Authorization is required.

Removing the mobile phone or email identifier is not currently allowed.
Attempting this will throw `undeletable_identifier`. For now, just replace the
existing identifier with the new one.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"type": "facebook",
	"value": "1122521981125307"
}
```

### `deauthenticate`

Revokes all refresh tokens and access tokens associated with the user, including
adding all non-expired access tokens to the JWT revocations Redis cache.
Authentication is required.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
