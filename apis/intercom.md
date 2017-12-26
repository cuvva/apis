# intercom service

Base URLs:

- dev: https://service-intercom.dev.ext.cuvva.co
- prod: https://service-intercom.prod.ext.cuvva.co
- sandbox: https://service-intercom.sandbox.ext.cuvva.co

## API

Conforms to the [Cuvva services standard][1]. However, this service is *not*
RESTful. Instead, it follows more of a simplified RPC approach.

All methods are specified below. Methods are called as
`POST /1/:version/:method`.

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

### `get_hash`

Generates the SHA256-HMAC digest required for Intercom's "secure mode":
https://developers.intercom.com/v2.0/docs/ios-identity-verification

#### Request

```json
{
	"client": "ios",
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

`client` can be `web`, `ios`, or `android`.

#### Response

```json
{
	"hash": "3aed6baac145ab7858417b1780a959e5f276165b1a3f6ce2e31122d667ca766a"
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
