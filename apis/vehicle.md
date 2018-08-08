# service-vehicle

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-vehicle
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-vehicle


## API

### Endpoints

#### Lookup Vehicle by ID

	GET /1/vehicles/[id:]<vehicle id>

Returns information about the vehicle referenced (including DVLA data) as referenced by our internal ID.

**Example Body:**

```json
{
	"id": "59f7152006a2300000000000",
	"example": false,
	"vrm": "LB07SEO",
	"prettyVrm": "LB07 SEO",
	"make": "Volkswagen",
	"model": "Polo",
	"variant": "SE 80",
	"color": "Silver"
}
```


#### Lookup Vehicle by VRM

	GET /1/vehicles/vrm:<vehicle vrm>

Returns information about the vehicle referenced (including DVLA data) as referenced by a VRM.

See [Lookup Vehicle by ID](#Lookup Vehicle By ID) for example responses.


#### List Example Vehicles

	GET /1/example_vehicles

Returns an array of vehicles which is used as the default garage contents.

All example vehicles purposefully have some fields left blank and an invalid ID, prefixed with `EXAMPLE-`, which will be unique in perpetuity.

**Example Body:**

```js
[
	{
		"id": "EXAMPLE-1",
		"vrm": "",
		"prettyVrm": "",
		"manufactureYear": 2004,
		"make": "Vauxhall",
		"model": "Corsa",
		"variant": "Design 16V",
		"color": "Silver",
		"example": true,
		"estimatedValue": 2000
	},
	...more_vehicles
]
```


### Error Codes

Error codes conform to the [Cuvva Standard](https://github.com/cuvva/standards), and may be wrapped in one or more encapsulating error object, where the real error exists as a reason.

- `ambiguous_data`: more than one vehicle exists for query
- `id_invalid`: given ID is not a valid MongoDB ObjectID
- `missing_vehicle_information`: vehicle exists but does not contain our minimum required information
- `not_found`: vehicle searched for was not found
- `vehicle_banned`: we (or at underwriters request) have specifically banned a vehicle
- `vin_incompatible`: vehicle exists but is currently incompatible with our system
- `vin_invalid`: given VIN is not a valid post-2000 VIN (as far as we are concerned)
- `vrm_invalid`: given VRM could not be coerced to a VRM we can lookup
- `vrm_mismatch`: VRM from data source does not match lookup
- `vrm_missing`: vehicle exists but contains no VRM
- `unknown`: an unknown error occurred, message suppressed and sent to sentry
