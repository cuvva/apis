# incident service

Base URLs:

- dev: https://api.dev.cuv-nonprod.app/1/service-incident
- prod: https://api.prod.cuv-prod.app/1/service-incident
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-incident

## Incident Categories

- `accident`: extra fields: `at_fault` (`yes`/`no`/`joint`) and `bodily_injury`
- `theft`: extra field: `theft_of` (`vehicle` or `contents`)
- `glass`
- `fire`
- `environmental`: includes floods, storm, etc.
- `malicious`: includes malicious damage, riots, etc.
- `misc`: other/unknown

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
Output is always JSON. All methods require authorization.

Where no response is specified for a method, no response will be provided. The
status code will be 204. The response may be changed to return data - with a
different 2xx status code - at any time. This would not be considered a breaking
change.

### `create_incidents`

Adds the array of incidents to the user. This *does not* replace existing
incidents - it appends new ones only.

Incidents which occurred more than five years ago will be rejected, along with
any incidents in the future.

For non-internal clients, if the user is not allowed to declare further
incidents, `incidents_locked` will be thrown. The user should contact customer
support to update their incidents.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"incidents": [
		{
			"date": "2014-01-01",
			"category": "accident",
			"cost": 500000,
			"at_fault": "joint",
			"bodily_injury": true
		},
		{
			"date": "2014-01-01",
			"category": "theft",
			"cost": 500000,
			"theft_of": "vehicle"
		},
		{
			"date": "2014-01-01",
			"category": "fire",
			"cost": 500000
		}
	]
}
```

### `clear_incidents`

Records that the user has had zero incidents over the last five years. This will
remove all existing incident records, if any.

For non-internal clients, if the user has any existing incidents,
`clearing_prohibited` will be thrown. The user should contact customer support
to clear their incidents.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

### `retrieve_lock_status`

Retrieve the details of how incident declarations are restricted.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

#### Response

```json
{
	"declarations_restricted": false
}
```

### `list_latest_incidents`

Lists the latest state of the user's incidents.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

#### Response

```js
{
	"incidents": [
		{
			"id": "ab017a8f-f6ad-453c-a6cc-37438744e923",
			"declared_at": "2017-10-01T14:12:19Z",
			"declared_late": false,
			"date": "2014-02-01",
			"category": "accident",
			"cost": 500000,
			"at_fault": "joint",
			"bodily_injury": true
		},
		{
			"id": "2264b46c-ecd1-48b3-a893-68431933b5e3",
			"declared_at": "2017-10-01T14:12:19Z",
			"declared_late": false,
			"date": "2014-01-01",
			"category": "theft",
			"cost": 500000,
			"theft_of": "vehicle"
		},
		{
			"id": "bbb45544-9d74-48e5-b020-6890f11b0fe3",
			"declared_at": "2017-11-08T10:52:00Z",
			"declared_late": true,
			"date": "2014-01-01",
			"category": "fire",
			"cost": 500000
		}
	]
}
```

The field `incidents` will be `null` if no data has been received yet.

[1]: https://github.com/cuvva/standards/blob/master/services.md
