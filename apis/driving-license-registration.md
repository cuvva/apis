# service-driving-license-registration

Base URLs:

- prod: https://service-proxy.prod.ext.cuvva.co/1/service-driving-license-registration/1
- dev: https://service-proxy.dev.ext.cuvva.co/1/service-driving-license-registration/1
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-driving-license-registration/1

## Versions

### 2018-05-01

This is the first documented version

## API

### `get_license`

If a license has not been registered against this user ID, the service will return the error `no_license_registered`.

#### Request

```json
{
	"user_id": "user_000000BQ3EYeeuOVLtYkgBdF77pZY",
	"revision": "dlrevent_000000BQ3TEDt7iRZdK1afwQfRCYS"
}
```

`revision` can be omitted if you want the latest revision.

#### Response

```json
{
	"revision": "dlrevent_000000BQ3TEDt7iRZdK1afwQfRCYS",
	"dln": {
		"value": "MILLE308039GE9HT",
		"verified": true
	},
	"postcode": {
		"value": "E1 1BJ",
		"verified": false
	},
	"full_license_override": {
		"activated_on": "2018-04-24T10:03:43Z",
		"expired": true
	}
}
```

Both `dln`.`verified` and `postcode`.`verified` will only be `false` if we were unable to verify them at the time of declaration. If they were verified as false at the time they were declared, the request to set them would fail.

The entire `postcode` object can be `null`.

`full_license_override` is `null` if:
- user has a full license
- there has never been an override in place
- there was an override and we found it to be true

### `declare_license`

This will throw if:
- the user already has a DLN attached to their account: `dln_already_attached`
- the DLN is used on any other account: `already_exists`
- the license is not found: `license_not_found`

#### Request

```json
{
	"user_id": "user_000000BQ3EYeeuOVLtYkgBdF77pZY",
	"dln": "MILLE308039GE9HT"
}
```

#### Response

```json
{
	"dln": {
		"value": "MILLE308039GE9HT",
		"verified": true
	}
}
```

`dln`.`verified` will only be `false` if we were unable to verify it at the time of declaration. If it was not found at the time it was declared, the request to set it would fail.

### `declare_test_pass`

It will throw if:
- they are already a full license holder: `already_passed`
- there is no DLN on this account:`no_license_registered`
- a test pass already exists: `test_pass_already_attached`

#### Request

```json
{
	"user_id": "user_000000BQ3EYeeuOVLtYkgBdF77pZY"
}
```

### `update_license`

This will throw if:
- you've never declared a license: `no_license_registered`
- you're changing the DLN and it already exists: `already_exists`
- the `postcode` does not match: `postcode_mismatch`
- the license is not valid: `invalid_license`
- you don't provide a DLN or postcode: `invalid_input`

The DLN can only be changed if:
- it's unverified
- user has never purchased

However the postcode can be changed at any time, subject to verification.

#### Request

```json
{
	"user_id": "user_000000BQ3EYeeuOVLtYkgBdF77pZY",
	"dln": "MILLE308039GE9HT",
	"postcode": "E11BJ"
}
```

`dln` and `postcode` can be individually omitted depending on what you're looking to update, however 1 must be present between the two.

#### Response

```json
{
	"dln": {
		"value": "MILLE308039GE9HT",
		"verified": true
	},
	"postcode": {
		"value": "E1 1BJ",
		"verified": true
	}
}
```
