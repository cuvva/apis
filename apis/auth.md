# service-auth

Base URLs:

- prod: https://service-auth.prod.ext.cuvva.co/2
- sandbox: https://service-auth.sandbox.ext.cuvva.co/2

## Versions

- 2017-05-09: first documented version - service already existed at this point
- 2017-07-24: no change - just needed for app compatibility
- 2017-11-23: no change - just needed for app compatibility

## Methods

### `register_user`

Creates the user with unverified identifiers (other than with social login tokens) and returns authentication data - including an access token, refresh token, etc.

If a user already exists for any of the identifiers, an `already_exists` error will be thrown. If the format or contents of any identifier value is invalid, `invalid_identifiers` will be thrown with various different inner errors depending on the exact reason.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"email": "james@cuvva.com",
	"mobile_phone": "+447700900123",
	"facebook_token": "EAAWgC7Fr0sABABA3gzOviDuZDZD",
	"google_token": "EAAWgC7Fr0sABABA3gzOviDuZDZD",
}
```

The `facebook_token`, `google_token` and `mobile_phone` fields are optional.

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

### `send_authorization_code`

Sends an authorization code to the email address provided.

This is equivalent to the "Authorization Endpoint", as per [section 3.1 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-3.1). However, it is **not** spec-compliant, as the process works via email instead.

Conforms to [section 4.3 of the PKCE spec](https://tools.ietf.org/html/rfc7636#section-4.3).

Authorization codes are valid for a short period of time. When another is requested, any existing unused codes are **left as-is** - allowing the user to use any of the emails. However, once any code is used, all existing codes become unusable.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"response_type": "code",
	"redirect_uri": "https://magic.cuvva.com/auth-callback",
	"state": "zCKoiZpLa8PcDZLJthr_Y_96pCrGtELDD-cY3XDPMxo",
	"code_challenge_method": "S256",
	"code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
	"email": "james@cuvva.com"
}
```

The `redirect_uri` must precisely match an expected value, as defined on the server-side. No parameters or components may be added.

The `state` and code verifier are nonces. A different value must be used for each, and the values must be regenerated for every request - never reuse them. The client must persist the nonces until the redirect URI has been used, or the expiry date has passed - whichever is earlier.

Generate the nonces with at least 32 bits of entropy from a cryptographically secure source of randomness. It is recommended to encode both as base64url strings (**standard base64 will not work**).

The only accepted `code_challenge_method` is `S256`.

The `email` field is not defined by the OAuth or PKCE specs, but specifies where the authorization code should be sent, obviously.

#### Response

```json
{
	"expires_in": 1800,
	"expires_at": "2018-02-09T16:55:36Z"
}
```

Normally, no response would be directly received for the "Authorization Endpoint". However, as the process works differently, the response provides some additional information to assist the client.

The `expires_at` field specifies when the authorization code sent to the user will expire, and the `expires_in` field is the number of seconds until that time.

The client should inform the user if the code expires before it has been used. Although the client may still receive the code, it will not be usable.

**Caution:** the local system time may be wrong, which could make it impossible to login with a naive client implementation. The client should prevent this by checking the local time against the `Date` header provided by the server.

#### Redirect

Once the user confirms the authorization, they will arrive at the redirect URI. The authorization response is provided via the query component of the URI.

```
https://magic.cuvva.com/auth-callback
	?code=c5c5f381833193cc71de42de.e2d09621646f6c104d7d6def9d1243e5fc22b0df765f8351495906c0ff2d0677
	&state=zCKoiZpLa8PcDZLJthr_Y_96pCrGtELDD-cY3XDPMxo
```

Whitespace added to the URI for the purposes of this documentation - no whitespace will be present during actual use.

The client must verify that the `state` parameter matches the one generated while making the original request. After checking this, the state must be discarded - so it can't be reused.

### `authenticate`

Creates and returns authentication data - access token, refresh token, etc.

This is the "Token Endpoint", as per [section 3.2 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-3.2).

The `access_token` returned is short lived. The `refresh_token` allows you to
get a new access token once the original one has expired. Refresh tokens last a
long time, but can only be used once - they are replaced when used.

#### Request

Multiple methods of authenticating are available. Specify via the `grant_type`:

##### `authorization_code`

Conforms to [section 4.1.3 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-4.1.3) and [section 4.5 of the PKCE spec](https://tools.ietf.org/html/rfc7636#section-4.5).

Once an authorization code has been successfully used, it cannot be reused, and any remaining unused codes are revoked.

**Caution:** once an authorization code has been used successfully, clients must ensure that the exchange process is not attempted again. Doing so will cause all resulting access tokens and refresh tokens to be revoked.

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"grant_type": "authorization_code",
	"redirect_uri": "https://magic.cuvva.com/auth-callback",
	"code": "c5c5f381833193cc71de42de.e2d09621646f6c104d7d6def9d1243e5fc22b0df765f8351495906c0ff2d0677",
	"code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"
}
```

##### `identifier_token`

Authenticates based on an external social login provider.

If the token is invalid, `invalid_token` will be thrown. If the token is valid
but no user exists for the identifier, `identifier_not_found` will be thrown.

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"grant_type": "identifier_token",
	"identifier_type": "facebook",
	"identifier_value": "1122521981125307",
	"identifier_token": "EAAWgC7Fr0sABABA3gzOviDuZDZD"
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

### `deauthenticate`

Revokes all authorization codes, refresh tokens and access tokens associated with the user, including adding all non-expired access tokens to the JWT revocations Redis cache. Authorization is required.

#### Request

```json
{
	"client_id": "06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f",
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
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
  "facebook_id": "1122521981125307",
  "google_id": "1122521981125307"
}
```

### `send_identifier_token`

Generates and sends a verification token to the given identifier. The token can then be used to replace the identifier of the given type on the account.

All requests must now be authorized - previously this could be used prior to login.

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

Removing the email and mobile phone identifiers is not currently allowed.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"type": "facebook",
	"value": "1122521981125307"
}
```
