# auth service

Base URL:

- https://auth-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `POST /1/verification_token_send`

Send a verification token with the given `method` (currently only `sms`).

No authentication/authorization is required to use this method.

Example request body:

```json
{
	"method": "sms",
	"identifier": "+447700900123"
}
```

Response will be a 204 (indicating no content). In the future, this may change
to a 200 with a JSON body without notice.

### `POST /1/oauth/token`

Issue an access token. This is the "Token Endpoint" as defined in the
[OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.2) and
[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) specs.

Currently, the only content type accepted is `x-www-form-urlencoded`, in line
with the OAuth spec. This will be expanded to also allow JSON eventually, but
presently JSON is not accepted.

No authentication/authorization is required to use this method.

Example request body:

```
client_id=06c92f2c-8bbb-403c-9ee7-3b3b8eb0b30f
&grant_type=verification_token
&verification_method=sms
&verification_identifier=%2B447700900123
&verification_token=969775
```

The line-breaks are for clarity only and must not be included in real requests.

Example response body:

```json
{
	"access_token": "01.eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkifQ.",
	"token_type": "bearer",
	"expires_in": 86399,
	"id_token": "eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkifQ."
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
