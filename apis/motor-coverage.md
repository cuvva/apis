# motor coverage service

Base URLs:

- dev: https://service-proxy.dev.ext.cuvva.co/1/service-motor-coverage
- prod: https://service-proxy.prod.ext.cuvva.co/1/service-motor-coverage
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-motor-coverage

## Old REST API (`/1`)

Conforms to the [Cuvva services standard][1].

Some monetary values are transmitted as pence, and some as pounds:

- excess amounts: pounds - `750` == 750 GBP
- valuations: pounds - `15000` == 15,000 GBP
- everything else: pence - `523` == 5.23 GBP

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
	},
	"recognizedTypes": ["duration", "end_of_day"]
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
	"productType": "learner",
	"quotes": [
		{
			"type": "end_of_day",
			"duration": 3600,
			"totalPremium": 459,
			"ipt": 44,
			"extraFees": 100,
			"vat": 20,
			"deductions": 0,
			"totalPayable": 623,
			"mandatoryExcess": 500,
			"payableId": "56d49c598211890100b12b4d",
			"underwriterId": "815facf0-cebb-4b63-98cd-7b63de91d465",
			"underwriterLegalName": "Foobar Insurance Company Limited",
			"underwriterShortName": "Foobar",
			"policyUrl": "https://example.com",
			"endDate": "2017-06-02T00:00:00Z",
			"endOfDayUpgrade": false,
			"declarations": [
				{
					"important": false,
					"code": "vehicle_mods",
					"text": "This vehicle has no modifications"
				}
			]
		}
	]
}
```

`productType` can currently be `unknown`, `standard`, `learner`, `van` or
`taxi`. Unrecognised values should be treated as `unknown`.

`declarations` can be empty, in which case none should be shown.

`type` is either `duration` or `end_of_day`. Where `duration` is a traditional
quote where duration is set. `end_of_day` is a quote where the duration is a
minimum and the policy issued will last until midnight.

`endDate` is a date where the policy will be expected to end.

`endOfDayUpgrade` is for when the quote given is an upgrade quote that'll last
until the end of the day. It specifically indicates the user has purchased a
policy already today and this is a quote to just 'top up' for the rest of the
day.

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
	},
	"recognizedTypes": ["duration", "end_of_day"]
}
```

Example response body:

```json
{
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"productType": "learner",
	"quotes": [
		{
			"type": "duration",
			"duration": 3600,
			"totalPremium": 459,
			"ipt": 44,
			"extraFees": 100,
			"vat": 20,
			"deductions": 0,
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
	"startDate": "2016-02-29T18:16:12Z",
	"endDate": "2016-02-29T19:16:12Z",
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"vehicleId": "56150f6fd0acdc14008e9551",
	"vehicleValue": 5000,
	"incidentPhone": "+447700900999",
	"detailsUrl": "https://example.com",
	"policyUrl": "https://example.com",
	"underwriterId": "815facf0-cebb-4b63-98cd-7b63de91d465",
	"underwriterLegalName": "Foobar Insurance Company Limited",
	"underwriterShortName": "Foobar",
	"referenceCode": "PQ37E6HEDC",
	"pricing": {
		"totalPremium": 459,
		"ipt": 44,
		"extraFees": 100,
		"vat": 20,
		"deductions": 0,
		"totalPayable": 623
	},
	"location": {
		"latitude": 55.9462597,
		"longitude": -3.2015704,
		"postalCodeFull": "EH3 9DR",
		"postalCodeShort": "EH3"
	}
}
```

## New RPC API (`/2`)

Conforms to the [Cuvva services standard][1]. However, this service is *not*
RESTful. Instead, it follows more of a simplified RPC approach.

All methods are specified below. Methods are called as
`POST /2/:version/:method`.

The `:version` should be set to the latest version at the point at which you
implemented your client code. Versions must not be mixed and matched - all
calls to this service must use the same version string.

Neither the query string, nor any URL parameters are used. Input is only
supported via the body. Only JSON is supported for input and output. All
methods require authorization.

Where no response is specified for a method, no response will be provided. The
status code will be 204. The response may be changed to return data - with a
different 2xx status code - at any time. This would not be considered a breaking
change.

Monetary values are always in pence - never pounds. Percentages/rates are in
basis points (10,000 means 100%).

### Versions

Notable and breaking changes for each version are as follows:

- `2017-11-23`: the first version of the RPC-style API

### `list_events`

Retrieves the user's history of driving policy activity as a stream of events.

Events will be returned in chronological order (oldest first). The timestamp
relates to the point at which the event became known and available in the API.
It is intended to assist with reliable synchronization and ordering, rather
than being used to determine the point at which any given event actually took
place.

The number of events returned will be limited arbitrarily. As such, to get a
full history of events, requests must continue to be made with new `after_date`
inputs until an empty array is returned.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"after_date": "2015-01-01T00:00:00.000Z"
}
```

`after_date` is optional - omitting it will return the earliest events.

#### Response

```json
[
	{
		"type": "policy_financial_transaction",
		"timestamp": "2017-12-17T18:16:11.101Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:payment",
		"payload": {
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"pricing": {
				"total_premium": 1948,
				"ipt": 234,
				"ipt_rate": 1200,
				"extra_fees": 429,
				"vat": 0,
				"deductions": 0,
				"total_payable": 2611
			}
		}
	},
	{
		"type": "policy_created",
		"timestamp": "2017-12-17T18:16:11.123Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:creation",
		"payload": {
			"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"original_policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"reference_code": "PQ37E6HEDC",
			"start_date": "2017-12-17T12:54:38Z",
			"end_date": "2017-12-17T15:54:38Z",
			"incident_phone": "+447700900999",
			"vehicle": {
				"id": "56150f6fd0acdc14008e9551",
				"estimated_value": 500000
			},
			"documents": {
				"certificate_url": "https://example.com",
				"terms_url": "https://example.com"
			},
			"underwriter": {
				"id": "815facf0-cebb-4b63-98cd-7b63de91d465",
				"legal_name": "Foobar Insurance Company Limited",
				"short_name": "Foobar"
			},
			"start_location": {
				"latitude": 51.5238914467717,
				"longitude": -0.0838015764693966,
				"postal_code_full": "EC2A 4DP",
				"postal_code_short": "EC2A"
			}
		}
	},
	{
		"type": "policy_cancelled",
		"timestamp": "2017-12-19T01:45:58.942Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:cancellation-0",
		"payload": {
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"type": "early_end",
			"new_end_date": "2017-12-17T14:54:38Z"
		}
	},
	{
		"type": "policy_financial_transaction",
		"timestamp": "2017-12-19T01:45:58.952Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:refund-0",
		"payload": {
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"pricing": {
				"total_premium": -649,
				"ipt": -78,
				"ipt_rate": 1200,
				"extra_fees": -143,
				"vat": 0,
				"deductions": 0,
				"total_payable": -870
			}
		}
	},
	{
		"type": "policy_financial_transaction",
		"timestamp": "2017-12-25T16:02:11.825Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:refund-1",
		"payload": {
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"pricing": {
				"total_premium": -1299,
				"ipt": -156,
				"ipt_rate": 1200,
				"extra_fees": -286,
				"vat": 0,
				"deductions": 0,
				"total_payable": -1741
			}
		}
	},
	{
		"type": "policy_cancelled",
		"timestamp": "2017-12-25T16:02:11.829Z",
		"unique_key": "policy:9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855:cancellation-1",
		"payload": {
			"policy_id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"type": "void",
			"new_end_date": null
		}
	}
]
```

`unique_key` exists to assist with deduplication. It must be treated as an
opaque string - do not attempt to parse it.

"Dependent" events will not always be in the perfect order. For example, as
shown in above, the policy payment event occurred a few milliseconds prior to
the policy creation event.

### `list_active_policy_ids`

Retrieves a list of currently-active policy IDs.

Primarily useful for searching through the event stream without having to rely
on local time accuracy.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159"
}
```

#### Response

```json
[
	"9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855"
]
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
