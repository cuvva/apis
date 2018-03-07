# flexi service

Base URLs:

- dev: https://service-proxy.dev.ext.cuvva.co/1/service-flexi
- prod: https://service-proxy.prod.ext.cuvva.co/1/service-flexi
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-flexi

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

### `generate_quote`

#### Request

```json
{
	"user_id": "1f94f1fe-48e4-4e7d-a587-df82a940bbb5",
	"start_date": "2016-10-25",
	"vehicle": {
		"id": "56151ccad0acdc14008e95c3",
		"estimated_value": 1000000,
		"owner_code": "P",
		"registered_keeper_code": "P",
		"overnight_storage": {
			"category_code": "4",
			"location": {
				"latitude": 51.5192945,
				"longitude": -0.0573541
			}
		}
	},
	"claims_bonus": {
		"years": 4
	},
	"location": {
		"latitude": 51.5192945,
		"longitude": -0.0573541
	},
	"recognized_types": ["duration", "end_of_day"]
}
```

The start date here can be provided as `null` meaning now. If provided, the
start date must be between 1 day and 7 days from the current date.

The start date CAN be passed as today, however it is a special case. Passing the
start date as today is the equivalent of passing in `null`.

Location here is used to generate an hourly estimate (with conditional
estimate if needed), it is also passed onto our fraud system. It is the physical
location of the client device.

#### Response

```json
{
	"id": "73c5fcaa-95c6-4784-aebd-0f729c1e5373",
	"user_id": "6ca2640f-f0bd-4767-a6e1-def4811f6f12",
	"vehicle_id": "56151ccad0acdc14008e95c3",
	"expiry_date": "2016-07-27T10:02:53.002Z",
	"mandatory_excess": 50000,
	"payable_id": "56d49c598211890100b12b4d",
	"underwriter_id": "815facf0-cebb-4b63-98cd-7b63de91d465",
	"underwriter_legal_name": "Foobar Insurance Company Limited",
	"underwriter_short_name": "Foobar",
	"policy_url": "https://example.com",
	"declarations": [
		"vehicle_performance_mods",
		"vehicle_impounded",
		"vehicle_use_code_01",
		"user_criminal_convictions",
		"user_medical_conditions",
		"user_policy_voided"
	],
	"underlying_pricing": {
		"new": {
			"total_premium": 861,
			"ipt": 34,
			"extra_fees": 100,
			"vat": 20,
			"deductions": 500,
			"total_payable": 515,
			"discount": {
				"type": "flat",
				"value": 500,
				"original": {
					"total_premium": 861,
					"ipt": 34,
					"extra_fees": 100,
					"vat": 20,
					"deductions": 0,
					"total_payable": 1015
				}
			}
		},
		"conditional": null
	},
	"hourly_average_estimated_pricing": {
		"new": 515,
		"conditional": 415
	},
	"hourly_estimates": [
		{
			"type": "duration",
			"duration": 3600,
			"current": null,
			"conditional": null,
			"new": 411
		},
		{
			"type": "end_of_day",
			"duration": 86400,
			"current": null,
			"conditional": null,
			"new": 1233
		}
	],
	"recognized_types": ["duration", "end_of_day"]
}
```

### `generate_estimate`

Generate an estimated quote for a Flexi subscription.

#### Request

```json
{
	"user_id": "1f94f1fe-48e4-4e7d-a587-df82a940bbb5",
	"vehicle": {
		"id": "56151ccad0acdc14008e95c3",
		"overnight_storage": {
			"category_code": null,
			"location": {
				"latitude": 51.5192945,
				"longitude": -0.0573541
			}
		},
		"estimated_calue": null,
		"owner_code": null,
		"registered_keeper_code": null
	},
	"claims_bonus": {
		"years": 4
	},
	"location": {
		"latitude": 51.5192945,
		"longitude": -0.0573541
	}
}
```

All rules regarding the `generate_quote` request apply here as well.

Location here is used to generate an hourly estimate (with conditional
estimate if needed), it is also passed onto our fraud system. It is the
physical location of the client device.

#### Response

```json
{
	"user_id": "1f94f1fe-48e4-4e7d-a587-df82a940bbb5",
	"underlying_pricing": {
		"duration": 2419200,
		"total_payable": 4055
	},
	"hourly_average_pricing": {
		"total_payable": 288
	},
	"hourly_estimates": [
		{
			"type": "duration",
			"duration": 3600,
			"current": null,
			"conditional": null,
			"new": 288
		},
		{
			"type": "end_of_day",
			"duration": 86400,
			"current": null,
			"conditional": null,
			"new": 864
		}
	]
}
```

No fields here are optional or omitted in the response.

### `get_profile`

Get a users profile, including Flexi specific information such as occupation,
residential address, and employment code.

#### Request

```json
{
	"user_id": "6ca2640f-f0bd-4767-a6e1-def4811f6f12"
}
```

#### Response

```json
{
	"id": "6ca2640f-f0bd-4767-a6e1-def4811f6f12",
	"occupation_code": "E",
	"employment_code": "A2",
	"residential_address": {
		"lines": [
			"42 Billings Street"
		],
		"locality": "Hamgate",
		"postal_code": "E1 1JB",
		"country_code": "GB"
	}
}
```

Some fields will always be present, although some, and perhaps all, many be set
to `null` if the  user has not yet completed their flexi profile.

A profile is considered complete if the `employment_code`, and
`residential_address` fields are set. `occupation_code` will only be present
with an appropriate `employment_code`.

### `set_new_details`

Update details regarding Flexi, specifically sending only Flexi related
information by profile or by subscription.

#### Request

```json
{
	"user_id": "6ca2640f-f0bd-4767-a6e1-def4811f6f12",
	"profile": {
		"employment_code": "E",
		"occupation_code": "052"
	},
	"subscriptions": [
		{
			"subscription_id": "5798d8d7544f3c0001c82656",
			"owner_code": "P",
		}
	],
	"location": {
		"latitude": 51.5192945,
		"longitude": -0.0573541
	}
}
```

It is important to note that any attribute on the profile object may be omitted.
Providing the value `null` will indicate the value should be removed, where as
omitting the value will leave it unchanged. Most keys can not be set to `null`.

The `user_id` field is always required.

Location here is used to generate an hourly estimate (with conditional
estimate if needed), it is also passed onto our fraud system. It is the physical
location of the client device.

#### Response

If the request can be processed without re-quoting, then a `204` response code
will be given and no further action is required.

The error code which will be shown in this case will be `repricing_required`.
Which means that all user subscriptions have changed pricing and the user will
have to accept the new prices before continuing.

```json
{
	"code": "repricing_required",
	"meta": {
		"confirmation_id": "5798e277ded1750001b67918",
		"change_fee": 0,
		"subscriptions": [
			{
				"subscription_id": "5798d8d7544f3c0001c82656",
				"vehicle_id": "56151ccad0acdc14008e95c3",
				"declarations": [
					{
						"important": false,
						"code": "a_declaration_code",
						"text": "A long text string you can display directly to a customer."
					}
				],
				"underlying_pricing": {
					"current": {
						"total_premium": 1122,
						"ipt": 112,
						"extra_fees": 100,
						"vat": 0,
						"deductions": 0,
						"total_payable": 1334
					},
					"new": {
						"total_premium": 1450,
						"ipt": 145,
						"extra_fees": 100,
						"vat": 0,
						"deductions": 0,
						"total_payable": 1695
					},
					"conditional": null
				},
				"hourly_average_estimated_pricing": {
					"current": 515,
					"new": 515,
					"conditional": 415
				},
				"hourly_estimates": [
					{
						"type": "duration",
						"duration": 3600,
						"total_payable": 587
					}
					{
						"type": "end_of_day",
						"duration": 86400,
						"total_payable": 2818
					}
				]
			}
		]
	}
}
```

All fields will be present here, even the `subscriptions` array will always have
at least 1 item inside the array.

### `confirm_new_details`

#### Request

```json
{
	"confirmation_id": "5798e277ded1750001b67918",
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3"
}
```

### `list_active_subscriptions`

#### Request

```json
{
	"user_id": "6ca2640f-f0bd-4767-a6e1-def4811f6f12"
}
```

#### Response

```json
[
	{
		"id": "5798d8d7544f3c0001c82656",
		"state": "active",
		"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
		"vehicle_id": "56151ccad0acdc14008e95c3",
		"start_date": "2016-07-01T10:25:29.002Z",
		"end_date": null,
		"next_payment_date": "2016-07-25",
		"recurring": true,
		"estimated_vehicle_value": 500000,
		"owner_code": "P",
		"registered_keeper_code": "P",
		"overnight_storage": {
			"category_code": "E",
			"location": {
				"latitude": 51.525507,
				"longitude": -0.058714
			}
		},

	},
	{
		"id": "5799132288e3ac00019008c8",
		"state": "active",
		"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
		"vehicle_id": "56151ccad0acdc14008e95c3",
		"start_date": "2016-07-03T11:49:19.252Z",
		"end_date": "2016-07-31T23:59:99.999Z",
		"next_payment_date": null,
		"recurring": false,
		"estimated_vehicle_value": 300000,
		"owner_code": "P",
		"registered_keeper_code": "P",
		"overnight_storage": {
			"category_code": "E",
			"location": {
				"latitude": 51.525507,
				"longitude": -0.058714
			}
		}
	}
]
```

In the case the user has no flexi subscriptions, the server will return an empty
array `[]`.

### `get_subscription`

The the full information set for a specific subscription.

#### Request

```json
{
	"subscription_id": "5798d8d7544f3c0001c82656",
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3"
}
```

#### Response

```json
{
	"id": "5798d8d7544f3c0001c82656",
	"state": "active",
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
	"vehicle_id": "56151ccad0acdc14008e95c3",
	"start_date": "2016-07-01T10:25:29.002Z",
	"end_date": null,
	"next_payment_date": "2016-07-25T09:00:00.000Z",
	"recurring": true,
	"estimated_vehicle_value": 500000,
	"owner_code": "P",
	"registered_keeper_code": "P",
	"overnight_storage": {
		"category_code": "E",
		"location": {
			"latitude": 51.525507,
			"longitude": -0.058714
		}
	},
	"claims_bonus": {
		"years": 2,
		"status": "pending"
	},
	"underlying_policies": [
		{
			"id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"underwriter_id": "815facf0-cebb-4b63-98cd-7b63de91d465",
			"underwriter_legal_name": "Foobar Insurance Company Limited",
			"underwriter_short_name": "Foobar",
			"start_date": "2016-07-01T10:25:29.002Z",
			"end_date": "2016-07-31T23:59:59.999Z",
			"incident_phone": "+447700900999",
			"certificate_url": "https://example.com",
			"policy_url": "https://example.com",
			"reference_code": "PQ37E6HEDC",
			"pricing": {
				"total_premium": 459,
				"ipt": 44,
				"extra_fees": 100,
				"vat": 20,
				"deductions": 0,
				"total_payable": 623,
				"discount": null
			}
		}
	]
}
```

You may also optionally pass a `revision_id` in to the request to find a
subscription at a specific point.

`end_date`: can be null.
`underlying_policies`: will always be an array with at least 1 item inside.
`underlying_policies`: include expired policies, and future policies, and
cancelled policies.
`discount` inside `underlying_policies`: can be null, all other values must
be set as values. If `discount` is set, it will be an object identical to the
output of the motor coverage service.

`claimsBonus.status` can be set to:
- `verified`: the NCB has been verified.
- `pending`: the NCB has not yet been verified but is waiting.
- `denied`: the NCB has been denied and needs retaking or removing.

### `create_subscription`

#### Request

```json
{
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
	"vehicle_id": "56151ccad0acdc14008e95c3",
	"quote_id": "73c5fcaa-95c6-4784-aebd-0f729c1e5373",
	"payment_id": "56d49c598211890100b12b4d",
	"claims_bonus": {
		"photo_id": "579912317640830001e8b8a5"
	}
}
```

`claims_bonus` as an entire object can be `null` if the `ncbYears` in previous
requests is set to 0.

#### Response

```json
{
	"id": "5798d8d7544f3c0001c82656",
	"state": "active",
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
	"vehicle_id": "56151ccad0acdc14008e95c3",
	"start_date": "2016-07-01T10:25:29.002Z",
	"end_date": null,
	"next_payment_date": "2016-07-25",
	"recurring": true,
	"estimated_vehicle_value": 500000,
	"owner_code": "P",
	"registered_keeper_code": "P",
	"overnight_storage": {
		"category_code": "E",
		"location": {
			"latitude": 51.525507,
			"longitude": -0.058714
		}
	},
	"claims_bonus": {
		"years": 2,
		"status": "pending"
	},
	"underlying_policies": [
		{
			"id": "9c3ce7e8-ddbb-4caf-93bf-b7ba9cd28855",
			"underwriter_id": "815facf0-cebb-4b63-98cd-7b63de91d465",
			"underwriter_legal_name": "Foobar Insurance Company Limited",
			"underwriter_short_name": "Foobar",
			"start_date": "2016-07-01T10:25:29.002Z",
			"end_date": "2016-07-31T23:59:59.999Z",
			"incident_phone": "+447700900999",
			"certificate_url": "https://example.com",
			"policy_url": "https://example.com",
			"reference_code": "PQ37E6HEDC",
			"pricing": {
				"total_premium": 459,
				"ipt": 44,
				"extra_fees": 100,
				"vat": 20,
				"deductions": 0,
				"total_payable": 623,
				"discount": null
			}
		}
	]
}
```

All values here will be present as show, `end_date` will always be `null`, there
will always be 1 policy in the `underlying_policies` array.

Within that 1 policy, `discount` will be null, or an object depending, identical
to the motor coverage service.

### `update_subscription_recurrence`

#### Request

```json
{
	"subscription_id": "5798d8d7544f3c0001c82656",
	"user_id": "d027f7d9-5dcb-4f6e-9f5c-00990b91b2c3",
	"recurring": true,
	"location": {
		"latitude": 51.5192945,
		"longitude": -0.0573541
	}
}
```

Location here is used to generate an hourly estimate (with conditional
estimate if needed), it is also passed onto our fraud system. It is the physical
location of the client device.

### `quote_cancellation`

#### Request

```json
{
	"subscription_id": "5798d8d7544f3c0001c82656",
	"user_id": "1f94f1fe-48e4-4e7d-a587-df82a940bbb5"
}
```

#### Response

```json
{
	"confirmation_id": "584afd0ea235f786095fa162",
	"terms": "A long string of 'legalese'",
	"expiry_date": "2016-12-09T19:23:22.023Z",
	"refund": {
		"reversed_total_premium": 861,
		"reversed_ipt": 34,
		"reversed_extra_fees": 100,
		"reversed_vat": 20,
		"reversed_deductions": 500,
		"reversed_total_payable": 515
	}
}
```

### `confirm_cancellation`

#### Request

```json
{
	"confirmation_id": "584afd0ea235f786095fa162",
	"user_id": "1f94f1fe-48e4-4e7d-a587-df82a940bbb5"
}
```

#### Response

```json
{
	"end_date": "2016-12-09T19:04:22.034Z",
	"type": "early_end",
	"terms": "A long string of 'legalese'"
}
```

Please note, the `terms` key returned here may be different from the terms used
in previous calls.

The `type` field can be one of the following values:
- `cancelled`: the policy was cancelled before inception (zero on-risk time)
- `early_end`: the policy was cancelled after inception (some on-risk time)

[1]: https://github.com/cuvva/standards/blob/master/services.md
