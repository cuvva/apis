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

Where no response is specified for a method, no response will be provided. The
status code will be 204. The response may be changed to return data - with a
different 2xx status code - at any time. This would not be considered a breaking
change.

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

### `report_potential_ownership`

Records potential vehicle ownership against a specific user ID.

#### Request

```json
{
	"userId": "13feab59-f669-4b6c-888a-077aaab235b4",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"estimatedValue": 5555
}
```

### `report_vague_potential_ownership`

Records potential vehicle ownership against a name and a set of identifiers.

#### Request

```json
{
	"vehicleId": "56150f6fd0acdc14008e9551",
	"estimatedValue": 5555,
	"name": "Freddy Macnamara",
	"identifiers": [
		{
			"type": "mobile_phone",
			"value": "+447700900123"
		},
		{
			"type": "email",
			"value": "freddy@cuvva.co"
		}
	]
}
```

The `name` field is optional, but strongly preferred. At least one identifier
must be provided. Multiple identifiers of the same type can be provided.

### `list_potentially_owned_vehicles`

Retrieves a set of vehicles which may be owned by the given user. At present,
this list is provided without any particular order.

#### Request

```json
{
	"userId": "13feab59-f669-4b6c-888a-077aaab235b4"
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

### `deny_ownership`

Flags a given user's potential ownership of a vehicle as false, thereby removing
it from the output of the list in the method above.

#### Request

```json
{
	"userId": "13feab59-f669-4b6c-888a-077aaab235b4",
	"vehicleId": "56150f6fd0acdc14008e9551"
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
