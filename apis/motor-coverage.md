# motor coverage service

Base URLs:

- dev: https://motor-coverage-dev.env.cuvva.co
- prod: https://motor-coverage-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `POST /1/quotes`

Generates a set of official quote offers.

The `originalPolicyId` field can be set to an existing active policy in order to
get quotes for extensions. Of course, the vehicle details cannot be different
for extensions.

For non-internal requests, the `userId` field must match the access token JWT
subject.

Example request body:

```json
{
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"originalPolicyId": null,
	"vehicleId": "56150f6fd0acdc14008e9551",
	"vehicleValue": 5000,
	"location": {
		"latitude": 55.9462597,
		"longitude": -3.2015704
	}
}
```

Example response body:

```json
{
	"id": "73c5fcaa-95c6-4784-aebd-0f729c1e5373",
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"vehicleValue": 5000,
	"expiryDate": "2016-02-29T12:10:30.354Z",
	"quotes": [
		{
			"duration": 3600,
			"totalPremium": 459,
			"ipt": 44,
			"extraFees": 100,
			"vat": 20,
			"totalPayable": 623,
			"mandatoryExcess": 500,
			"payableId": "56d49c598211890100b12b4d",
			"underwriterId": "815facf0-cebb-4b63-98cd-7b63de91d465",
			"underwriterLegalName": "Foobar Insurance Company Limited",
			"underwriterShortName": "Foobar",
			"keyFactsUrl": "https://example.com",
			"policyUrl": "https://example.com"
		}
	]
}
```

### `POST /1/quotes/estimate`

Generates a set of estimates (*not* official quotes).

The `vehicleValue` field is optional for estimates, but all other fields must be
provided.

Estimates cannot be retrieved for extension quotes.

For non-internal requests, the `userId` field must match the access token JWT
subject.

Example request body:

```json
{
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"vehicleValue": 5000,
	"location": {
		"latitude": 55.9462597,
		"longitude": -3.2015704
	}
}
```

Example response body:

```json
{
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"quotes": [
		{
			"duration": 3600,
			"totalPremium": 459,
			"ipt": 44,
			"extraFees": 100,
			"vat": 20,
			"totalPayable": 623
		}
	]
}
```

### `POST /1/policies`

Creates a policy.

The `vehiclePhotoId` field is not provided when extending an existing active
policy.

For non-internal requests, the owner of the specified quote must match the
access token JWT subject.

Example request body:

```json
{
	"quoteId": "73c5fcaa-95c6-4784-aebd-0f729c1e5373",
	"paymentId": "56d4a5433e7c6201004c8eee",
	"duration": 3600,
	"vehiclePhotoId": "56d4a555b28a970100e07f02"
}
```

Example response body:

```json
{
	"id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
	"duration": 3600,
	"startDate": "2016-02-29T18:16:12Z",
	"endDate": "2016-02-29T19:16:12Z",
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"vehicleValue": 5000,
	"incidentPhone": "+447700900999",
	"detailsUrl": "https://example.com",
	"keyFactsUrl": "https://example.com",
	"policyUrl": "https://example.com",
	"underwriterId": "815facf0-cebb-4b63-98cd-7b63de91d465",
	"underwriterLegalName": "Foobar Insurance Company Limited",
	"underwriterShortName": "Foobar",
	"referenceCode": "PQ37E6HEDC"
}
```

### `GET /1/users/:user_id/active_policies`

Retrieves a set of the user's current policy chains.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example response body:

```json
[
	{
		"originalPolicyId": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
		"currentPolicyId": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
		"policies": [
			{
				"id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
				"duration": 3600,
				"startDate": "2016-02-29T18:16:12Z",
				"endDate": "2016-02-29T19:16:12Z",
				"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
				"vehicleId": "56150f6fd0acdc14008e9551",
				"vehicleValue": 5000,
				"incidentPhone": "+447700900999",
				"detailsUrl": "https://example.com",
				"keyFactsUrl": "https://example.com",
				"policyUrl": "https://example.com",
				"underwriterId": "815facf0-cebb-4b63-98cd-7b63de91d465",
				"underwriterLegalName": "Foobar Insurance Company Limited",
				"underwriterShortName": "Foobar",
				"referenceCode": "PQ37E6HEDC"
			}
		]
	}
]
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
