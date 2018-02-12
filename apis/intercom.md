# service-intercom

Base URLs:

- dev: https://service-intercom.dev.ext.cuvva.co/1
- prod: https://service-intercom.prod.ext.cuvva.co/1
- sandbox: https://service-intercom.sandbox.ext.cuvva.co/1

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
