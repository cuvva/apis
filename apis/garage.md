# garage service

Base URLs:

- dev: https://garage-dev.env.cuvva.co
- prod: https://garage-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1]. This service is *not* RESTful, but
instead uses the simplified RPC-style interface first used in the
[v2 auth service](https://github.com/cuvva/service-auth#api).

All methods are specified below. Methods are called as
`POST /2/:version/:method`.

The `:version` is replaced with the date you implemented the code using the API.
Versions must not be mixed and matched - all calls to this service must use the
same version string.

Neither the query string, nor any URL parameters are used. Input is only
supported via the body. JSON and urlencoded forms are supported for input.
Output is always JSON. All methods require authorization.

### `get_suggested_vehicles`

Retrieves up to five vehicles which the user is likely to want to interact with,
ordered by relevancy where the most relevant vehicles come first. Sometimes zero
vehicles will be returned, often fewer than five.

The location data is not currently used, but is required in anticipation that we
will track the current location of vehicles in the future.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"location": {
		"latitude": 51.521,
		"longitude": -0.052
	}
}
```

#### Response

```json
[
	{
		"id": "56150f6fd0acdc14008e9551",
		"estimated_value": 5555,
		"vrm": "LB07SEO",
		"pretty_vrm": "LB07 SEO",
		"manufacture_year": 2007,
		"make": "Volkswagen",
		"model": "Polo",
		"color": "Silver"
	}
]
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
