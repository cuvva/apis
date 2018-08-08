# service-intercom

Base URLs:

- dev: https://api.dev.cuv-nonprod.app/1/service-intercom/1
- prod: https://api.prod.cuv-prod.app/1/service-intercom/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-intercom/1

## Versions

- 2017-05-09: first documented version - service already existed at this point
- 2017-07-24: no change - just needed for app compatibility
- 2017-11-23: no change - just needed for app compatibility

## Methods

### `get_hash`

Returns the relevant HMAC digest for use with Intercom's "secure mode".

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client": "ios"
}
```

#### Response

```json
{
	"hash": "c5f797ad49aed2d46c3c6a3f4e3a29e9ab17bdf28db944eee499861aeb22e09e"
}
```
