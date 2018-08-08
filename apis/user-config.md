# service-user-config

Base URLs:

- dev: https://api.dev.cuv-nonprod.app/1/service-user-config/1
- prod: https://api.prod.cuv-prod.app/1/service-user-config/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-user-config/1

## Versions

- 2018-06-07: service created

## Methods

### `get_config`

Provides config for the given user and client.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client": {
		"platform": "ios",
		"version": "1.2.3",
		"build": "1234"
	}
}
```

#### Response

```json
{
	"flags": [
		"some_feature_enabled"
	],
	"config": {
		"foo": "bar",
		"blah": true,
		"abc": 1,
		"123": {
			"foo": [
				{ "aaa": "bbb" }
			]
		}
	}
}
```

The values of the `config` object can be any JSON type, including `null`.
