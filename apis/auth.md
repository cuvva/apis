# service-auth

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-auth/2
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-auth/2

## Versions

### 2017-05-09

This is the first documented version - service already existed at this point.

To be withdrawn imminently. So far, the following has been changed:

- `register_user` has been removed
- `authenticate` no longer allows the use of email identifier tokens (6 digit codes)

### 2017-07-24

Alias of `2017-05-09` - just needed for app compatibility.

### 2017-11-23

Alias of `2017-05-09` - just needed for app compatibility.

### 2018-03-06

- `register_user` has been replaced completely - it now accepts a set of grants which represent identifiers
- `authenticate` no longer accepts email/mobile phone identifier tokens, but social login identifier tokens are still fine
- `send_identifier_token` requires authentication (as it is only usable with `replace_identifier` now)
- `replace_identifier` requires the `client_id` field
- sent identifier tokens are now user-specific, so ones sent by older versions cannot be used with this version, and vice-versa

Non-breaking but significant:

- `send_authorization_code` added
- `authenticate` now accepts authorization codes

### 2018-12-11

- `replace_identifier` removed
- `include_removed_identifiers` property now required in `get_user` and `get_user_by_identifier` endpoints

### 2020-06-02

- `send_authorization_code` updated to have both `identifier_type` and `identifier_value` keys for specifying email or user id.

## Methods

### `register_user`

Creates the user with a set of identifiers (verified via OAuth grants) and returns authentication data - including an access token, refresh token, etc.

If it is not possible to generate and persist a device ID, omit the device property from the request body. It is acceptable to send `null` for the advertising ID (`ad_id`).

#### Request

```js
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"grants": [
		{
			"grant_type": "authorization_code",
			"code": "blah",
			"code_verifier": "blah"
		},
		...more
	],
	"device" {
		"platform": "ios",
		"ad_id": "6D92078A-8246-4BA4-AE5B-76104861E7DC",
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
}
```

#### Response

```json
{
	"access_token": "eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkiLCJhdWQiOiIwNmM5MmYyYy04YmJiLTQwM2MtOWVlNy0zYjNiOGViMGIzMGYifQ.",
	"token_type": "bearer",
	"expires_in": 3600,
	"expires_at": "2016-06-01T00:00:00Z",
	"refresh_token": "02.reftok_000000BRxPaWGu0xDiHFOktPnBtK9.0335fc54f030a9a476d210854f4cb1f5def99f64ea063b806fde65563feb0c86",
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM"
}
```

#### `already_exists` error

If a user already exists for any of the identifiers, an `already_exists` error will be thrown, but the grants will not be consumed.

This means it is possible to then immediately login with the existing grant parameters. However, it's important to understand the context and offer appropriate context and a logical option to help the user continue.

The `index` refers to the index of the input array of `grants`. The `user_key` helps to identify whether the grants are attached to the same user account or not. Although they do currently include the user ID, this may change and they should be treated as opaque strings.

```json
{
	"code": "already_exists",
	"meta": {
		"grants": [
			{
				"index": 0,
				"user_key": "user/user_000000BRxPaWGu0xDiHFOktPnBtK7"
			},
			{
				"index": 1,
				"user_key": "user/user_000000BRxPaWGu0xDiHFOktPnBtK7"
			}
		]
	}
}
```

### `send_authorization_code`

Sends an authorization code to the email address provided, or the primary email address of the user ID provided.

This is equivalent to the "Authorization Endpoint", as per [section 3.1 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-3.1). However, it is **not** spec-compliant, as the process works via email instead.

Conforms to [section 4.3 of the PKCE spec](https://tools.ietf.org/html/rfc7636#section-4.3).

Authorization codes are valid for a short period of time. When another is requested, any existing unused codes are **left as-is** - allowing the user to use any of the emails. However, once any code is used, all existing codes become unusable.

If it is not possible to generate and persist a device ID, omit the device property from the request body. It is acceptable to send `null` for the advertising ID (`ad_id`).

#### Request

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"response_type": "code",
	"redirect_uri": "https://magic.cuvva.com/auth-callback",
	"state": "zCKoiZpLa8PcDZLJthr_Y_96pCrGtELDD-cY3XDPMxo",
	"code_challenge_method": "S256",
	"code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
	"identifier_type": "email",
	"identifier_value": "james@cuvva.com",
	"device": {
		"platform": "ios",
		"ad_id": "6D92078A-8246-4BA4-AE5B-76104861E7DC",
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
}
```

The `redirect_uri` must precisely match an expected value, as defined on the server-side. No parameters or components may be added.

The `state` and code verifier are nonces. A different value must be used for each, and the values must be regenerated for every request - never reuse them. The client must persist the nonces until the redirect URI has been used, or the expiry date has passed - whichever is earlier.

Generate the nonces with at least 32 bits of entropy from a cryptographically secure source of randomness. It is recommended to encode both as base64url strings (**standard base64 will not work**).

The only accepted `code_challenge_method` is `S256`.

The two `identifier_*` fields are not defined by the OAuth or PKCE specs, but specify where the authorization code will be sent. `identifier_type` can also be `user_id`, in which case the email will be sent to the primary email identifier on the specified user, if any.

#### Response

```json
{
	"expires_in": 1800,
	"expires_at": "2018-02-09T16:55:36Z",
	"redacted_email": "j****s@cuvva.com"
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
	?code=authzcode_000000BRxPaWGu0xDiHFOktPnBtKA.e2d09621646f6c104d7d6def9d1243e5fc22b0df765f8351495906c0ff2d0677
	&state=zCKoiZpLa8PcDZLJthr_Y_96pCrGtELDD-cY3XDPMxo
```

Whitespace added to the URI for the purposes of this documentation - no whitespace will be present during actual use.

The client must verify that the `state` parameter matches the one generated while making the original request. After checking this, the state must be discarded - so it can't be reused.

### `create_client_code`

Creates a one-time-use code, allowing another client to authenticate as the user that generated it. Creating a new code will invalidate all existing codes for a user.

Scopes are optional. If they are not provided, the scopes of the passed in refresh token are used.

Using the same refresh token twice will cause all resulting access tokens, refresh tokens, and client codes to be revoked.

**The refresh token returned must be used going forward, as the existing one has been consumed.**

Scopes are optional. If they are not provided, the scopes of the refresh token are used.

#### Request

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"refresh_token": "02.reftok_000000BuumCRA7CUbG5DNSFOUBDQw.d6b58f06b495a433c0873fd17cd787d4db2c43ae08222e13b95835153efeebc2",
	"scope": "self:official_app",
	"device": {
		"platform": "web",
		"ad_id": null,
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
}
```

#### Response

```json
{
	"refresh_token": "02.reftok_000000BuwpRGSquSlG5jo1nBgAjdj.bb9e526dbbfb8ac5b50f1970621e59dcd7a6fcf2fb2dd68e873e45c566c67ff2",
	"code": "H798B9DE",
	"expires_at": "2020-05-18T12:44:20.591Z"
}
```

If the error `client_code_duplicate` is returned to the client, it's safe to just try again.

### `authenticate`

Creates and returns authentication data - access token, refresh token, etc.

This is the "Token Endpoint", as per [section 3.2 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-3.2).

Rejects with error code `secondary_account` if the resolved user ID corresponds to a secondary account.

The `access_token` returned is short lived. The `refresh_token` allows you to
get a new access token once the original one has expired. Refresh tokens last a
long time, but can only be used once - they are replaced when used.

If it is not possible to generate and persist a device ID, omit the device property from the request body. It is acceptable to send `null` for the advertising ID (`ad_id`).

#### Request

Multiple methods of authenticating are available. Specify via the `grant_type`:

##### `authorization_code`

Conforms to [section 4.1.3 of the OAuth spec](https://tools.ietf.org/html/rfc6749#section-4.1.3) and [section 4.5 of the PKCE spec](https://tools.ietf.org/html/rfc7636#section-4.5).

Once an authorization code has been successfully used, it cannot be reused, and any remaining unused codes are revoked.

**Caution:** once an authorization code has been used successfully, clients must ensure that the exchange process is not attempted again. Doing so will cause all resulting access tokens and refresh tokens to be revoked.

When using an authorization code from an email identifier, if it's the first email the user is verifying it will revoke all other tokens/codes for the user.

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"grant_type": "authorization_code",
	"redirect_uri": "https://magic.cuvva.com/auth-callback",
	"code": "authzcode_000000BRxPaWGu0xDiHFOktPnBtKA.e2d09621646f6c104d7d6def9d1243e5fc22b0df765f8351495906c0ff2d0677",
	"code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk",
	"device": {
		"platform": "ios",
		"ad_id": "6D92078A-8246-4BA4-AE5B-76104861E7DC",
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
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
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"grant_type": "refresh_token",
	"refresh_token": "02.reftok_000000BRxPaWGu0xDiHFOktPnBtK9.960b19318364fb3838b97695b27c36287fbd02d677131dfd41261f7d1bd3a62c",
	"device": {
		"platform": "ios",
		"ad_id": "6D92078A-8246-4BA4-AE5B-76104861E7DC",
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
}
```


##### `client_code`

Authenticates as the user that generated the client code. The scopes provided must be a subset of the resolved scopes saved when the code was created.

If the IP address of the request doesn't match the IP address that was used when the code was created this will fail with the error `client_code_ip_mismatch`. The meta of this error will contain the key `user_id`, which is the user id of the user that requested the code. You can use this to send a magic link to the user to continue the flow in an alternative way.

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"grant_type": "client_code",
	"client_code": "A8BNHE4D",
	"device": {
		"platform": "ios",
		"ad_id": "6D92078A-8246-4BA4-AE5B-76104861E7DC",
		"cuv_id": "mIJG3vSEu8hdx6ESaIoPjh6527jD1ZlEBsmNVEBkkbUg6ZU22J299e"
	}
}
```

#### Response

```json
{
	"access_token": "eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkiLCJhdWQiOiIwNmM5MmYyYy04YmJiLTQwM2MtOWVlNy0zYjNiOGViMGIzMGYifQ.",
	"token_type": "bearer",
	"expires_in": 3600,
	"expires_at": "2016-06-01T00:00:00Z",
	"refresh_token": "02.reftok_000000BRxPaWGu0xDiHFOktPnBtK6.0335fc54f030a9a476d210854f4cb1f5def99f64ea063b806fde65563feb0c86",
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"secondary_user_ids": []
}
```

### `deauthenticate`

Revokes all authorization codes, refresh tokens and access tokens associated with the user, including adding all non-expired access tokens to the JWT revocations Redis cache. Authorization is required.

#### Request

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7"
}
```

### `get_user`

Retrieves data for the user and returns it. Authorization is required.

#### Request

```json
{
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"include_removed_identifiers": false
}
```

#### Response (for example primary user with a secondary account)

```json
{
	"id": "user_000000Be3fvsYfQ133g0lUmHemOa9",
	"email": "freddy@cuvva.com",
	"email_verified": true,
	"mobile_phone": "+447700900123",
	"mobile_phone_verified": false,
	"primary_user_id": null,
	"secondary_user_ids": [
		"user_000000Be3flMeXtwMx7TeZAZDNv1d"
	],
	"identifiers": [
		{
			"id": "userident_000000BhBQpDxxoU1PseWR9h0bmOu",
			"type": "email",
			"value": "freddy@cuvva.com",
			"made_primary_at": "2016-01-01T00:00:00Z",
			"primary": true,
			"last_verified_at": "2016-01-01T00:00:00Z",
			"attached_at": "2016-01-01T00:00:00Z"
		},
		{
			"id": "userident_000000BhBQpDxxoU1PseWR9h0bmOv",
			"type": "email",
			"value": "freddy+2@cuvva.com",
			"made_primary_at": null,
			"primary": false,
			"last_verified_at": null,
			"attached_at": "2016-01-01T00:00:00Z"
		},
		{
			"id": "userident_000000BhBQpDxxoU1PseWR9h0bmOw",
			"type": "mobile_phone",
			"value": "+44447700900123",
			"made_primary_at": "2016-01-01T00:00:00Z",
			"primary": true,
			"last_verified_at": "2016-01-01T00:00:00Z",
			"attached_at": "2016-01-01T00:00:00Z"
		},
		{
			"id": "userident_000000BhBQpDxxoU1PseWR9h0bmOx",
			"type": "mobile_phone",
			"value": "+44447700900124",
			"made_primary_at": "2016-01-02T00:00:00Z",
			"primary": false,
			"last_verified_at": "2016-01-02T00:00:00Z",
			"attached_at": "2016-01-02T00:00:00Z"
		}
	]
}
```

### `send_identifier_token`

Generates and sends a verification token to the given identifier. The token can then be used to replace the identifier of the given type on the account.

All requests must now be authorized - previously this could be used prior to login.

#### Request

```json
{
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"type": "mobile_phone",
	"value": "+447700900123"
}
```

### `attach_identifier`

Associates a new identifer with a user ID. If no existing primary identifier exists, it will be made the primary identifier.

If a token is provided in the request body, it will be verified.

Rejects with error code `secondary_account` if the user ID corresponds to a secondary account.

Public clients can attach an email identifier without a token but only if the user has no previously verified email addresses on their account.

#### Request

```json
{
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"type": "mobile_phone",
	"value": "+447700900123",
	"token": "123456"
}
```

### `detach_identifier`

Removes an identifier from the given user. Authorization is required. Primary identifiers cannot be detached.

#### Request

```json
{
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"type": "mobile_phone",
	"value": "+447700900123"
}
```

### `set_primary_identifier`

Sets a provided identifier as the primary for the given type.

```json
{
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"type": "email",
	"value": "james+newprimary@cuvva.com"
}
```

### `verify_identifier`

Verifies the token and marks an existing identifier as verified.

#### Request

```json
{
	"user_id": "user_000000BRxPaWGu0xDiHFOktPnBtK7",
	"client_id": "client_000000BPG6PISgRjLvxD5rf7Bf0FM",
	"type": "mobile_phone",
	"value": "+447700900123",
	"token": "123456"
}
```

### `list_currently_associated_users`

Returns user IDs for any secondary users currently associated to the input ID, including the input ID itself.

e.g. If the input ID is a primary account it returns itself plus all secondary accounts. If the input ID is a secondary it will include itself, the primary user ID, and the secondary accounts of that primary.

#### Request

```json
{
	"user_id": "user_000000BZ52kE3sKwJrbAqcX0cB52e"
}
```

#### Response

```json
{
	"primary_user_id": "user_000000BaWYg2EObmPndxV5u8uODhY",
	"user_ids": [
		"user_000000BZ52kE3sKwJrbAqcX0cB52e",
		"user_000000BaWYg2EObmPndxV5u8uODhY",
		"user_000000BaWYho1esvggGPGb8ylx13g"
	]
}
```

### `list_user_associations`

Returns the primary user id and an array of all user association entities related to a given user to 2-degrees of separation (returns direct-relationships, and also any associations related to those).

`status` can be one of "complete", "pending", "failed".

#### Request

```json
{
	"user_id": "user_000000BZ52kE3sKwJrbAqcX0cB52e"
}
```

#### Response (for a secondary user)

```json
[
	{
		"created_at": "2019-01-10T13:25:41.259Z",
		"primary_user_id": "local_user_000000BZ52kE3sKwJrbAqcX0cB52e",
		"secondary_user_id": "local_user_000000BZ7Zs3dwEH0RF3dACnB2H56",
		"status": "complete"
	},
	{
		"created_at": "2018-12-08T13:25:41.259Z",
		"primary_user_id": "local_user_000000BZ52kE3sKwJrbAqcX0cB52e",
		"secondary_user_id": "local_user_000000BZ7Zs3dwEH0RF3dACnB2H56",
		"status": "complete"
	}
]
```

#### Response (for a primary user)

```json
[
	{
		"created_at": "2019-01-10T13:25:41.259Z",
		"primary_user_id": null,
		"secondary_user_id": "local_user_000000BZ7Zs3dwEH0RF3dACnB2H56",
		"status": "complete"
	},
	{
		"created_at": "2018-12-08T13:25:41.259Z",
		"primary_user_id": "local_user_000000BZ52kE3sKwJrbAqcX0cB52e",
		"secondary_user_id": "local_user_000000BZ7Zs3dwEH0RF3dACnB2H56",
		"status": "complete"
	}
]
```
