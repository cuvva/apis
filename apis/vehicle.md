# vehicle service

Base URL:

- https://vehicle-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/vehicles/:vehicle_key`

Returns information about the vehicle referenced (including DVLA data).

The `vehicle_key` is a key-value pair like `field:value` which identifies the
vehicle. Valid fields are `id`, `vrm` and `vin`. If no field is specified, `id`
is assumed.

e.g. different ways of looking up LB07 SEO:
- registration plate: `GET /1/vehicles/vrm:LB07SEO`
- VIN (serial number): `GET /1/vehicles/vin:WVWZZZ9NZ7Y235732`
- id: `GET /1/vehicles/56150f6fd0acdc14008e9551`
- id alternative: `GET /1/vehicles/id:56150f6fd0acdc14008e9551`

Example response body:

```json
{
	"id": "56150f6fd0acdc14008e9551",
	"quotable": true,
	"vrm": "LB07SEO",
	"prettyVrm": "LB07 SEO",
	"vin": "WVWZZZ9NZ7Y235732",
	"abi": 53563201,
	"group": 7,
	"manufactureYear": 2007,
	"imported": false,
	"scrapped": false,
	"make": "Volkswagen",
	"model": "Polo",
	"variant": "SE 80",
	"color": "Silver",
	"seatCount": 5
}
```

### `GET /1/example_vehicles`

Returns an array of vehicles which is used as the default garage contents.

Example response body:

```json
[
	{
		"id": "56150f6fd0acdc14008e9551",
		"quotable": true,
		"vrm": "LB07SEO",
		"prettyVrm": "LB07 SEO",
		"vin": "WVWZZZ9NZ7Y235732",
		"abi": 53563201,
		"group": 7,
		"manufactureYear": 2007,
		"imported": false,
		"scrapped": false,
		"make": "Volkswagen",
		"model": "Polo",
		"variant": "SE 80",
		"color": "Silver",
		"seatCount": 5
	}
]
```

### `GET /1/users/:user_id/vehicles`

Returns an array of vehicles in the user's garage, as above, but including the
estimated value stated by the user. The vehicles are ordered by putting the most
recently bumped at the top.

For non-internal requests, `user_id` must match the access token JWT subject.

Example response body:

```json
[
	{
		"id": "56150f6fd0acdc14008e9551",
		"estimatedValue": 5555,
		"quotable": true,
		"vrm": "LB07SEO",
		"prettyVrm": "LB07 SEO",
		"vin": "WVWZZZ9NZ7Y235732",
		"abi": 53563201,
		"group": 7,
		"manufactureYear": 2007,
		"imported": false,
		"scrapped": false,
		"make": "Volkswagen",
		"model": "Polo",
		"variant": "SE 80",
		"color": "Silver",
		"seatCount": 5
	}
]
```

### `POST /1/users/:user_id/vehicles`

Attaches the vehicle to the user - i.e. puts it in the user's "garage". Up to
100 vehicles may be added. Additionally, the estimated value must be a positive
integer and cannot exceed 1,000,000.

For non-internal requests, `user_id` must match the access token JWT subject.

Example request body:

```json
{
	"id": "56150f6fd0acdc14008e9551",
	"estimatedValue": 5555
}
```

Example response body:

```json
{
	"id": "56150f6fd0acdc14008e9551",
	"estimatedValue": 5555,
	"quotable": true,
	"vrm": "LB07SEO",
	"prettyVrm": "LB07 SEO",
	"vin": "WVWZZZ9NZ7Y235732",
	"abi": 53563201,
	"group": 7,
	"manufactureYear": 2007,
	"imported": false,
	"scrapped": false,
	"make": "Volkswagen",
	"model": "Polo",
	"variant": "SE 80",
	"color": "Silver",
	"seatCount": 5
}
```

### `POST /1/users/:user_id/vehicles/:vehicle_id/bump`

"Bumps" the vehicle to the top of the user's garage.

For non-internal requests, `user_id` must match the access token JWT subject.

### `DELETE /1/users/:user_id/vehicles/:vehicle_id`

Removes the vehicle from the user's garage.

For non-internal requests, `user_id` must match the access token JWT subject.

[1]: https://github.com/cuvva/standards/blob/master/services.md
