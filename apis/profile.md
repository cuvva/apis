# profile service

Base URLs:

- dev: https://service-profile.dev.ext.cuvva.co
- prod: https://service-profile.prod.ext.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/users/:user_id`

Returns the latest profile for the user.

For non-internal requests, `id` must match the access token JWT subject.

Example response body:

```json
{
	"id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T00:00:00Z",
	"title": "Mr",
	"familyName": "Billingham",
	"personalName": "James",
	"preferredName": "Bob",
	"sex": "M",
	"birthDate": "1990-01-01",
	"licenseNumber": "BILLI901010J99AA",
	"profilePhotoId": "56d4b4cdf96cf40100b6b6d0",
	"licensePhotoId": "56d4b4d241a58b01003e3419",
	"address": {
		"lines": [
			"Cuvva c/o Codebase",
			"3 Lady Lawson Street"
		],
		"locality": "Edinburgh",
		"postalCode": "EH3 9DR",
		"countryCode": "GB"
	},
	"claims": []
}
```

### `PUT /1/users/:user_id`

Update the profile, returns the new state of the profile.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example request body:

```json
{
	"address": {
		"lines": [
			"Cuvva c/o Barclays Accelerator",
			"69-89 Mile End Road"
		],
		"locality": "London",
		"postalCode": "E1 4TT",
		"countryCode": "GB"
	},
	"claims": [
		{
			"incidentDate": "2014-01-01",
			"category": "Accident",
			"bodilyInjury": false,
			"atFault": false,
			"value": 500
		}
	]
}
```

Example response body:

```json
{
	"id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T00:00:00Z",
	"title": "Mr",
	"familyName": "Billingham",
	"personalName": "James",
	"preferredName": "Bob",
	"sex": "M",
	"birthDate": "1990-01-01",
	"licenseNumber": "BILLI901010J99AA",
	"profilePhotoId": "56d4b4cdf96cf40100b6b6d0",
	"licensePhotoId": "56d4b4d241a58b01003e3419",
	"address": {
		"lines": [
			"Cuvva c/o Barclays Accelerator",
			"69-89 Mile End Road"
		],
		"locality": "London",
		"postalCode": "E1 4TT",
		"countryCode": "GB"
	},
	"claims": [
		{
			"incidentDate": "2014-01-01",
			"category": "Accident",
			"bodilyInjury": false,
			"atFault": false,
			"value": 500
		}
	]
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
