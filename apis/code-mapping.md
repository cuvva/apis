# service-code-mapping

Base URLs:

- prod: https://service-proxy.prod.ext.cuvva.co/1/service-code-mapping/1
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-code-mapping/1

## Versions

- 2018-06-07: service created

## Methods

### `get_error_codes`

Provides messages for all user-facing error codes.

#### Response

```js
{
	"codes": {
		"cannot_quote": {
			"title": "Oh no!",
			"message": "We couldn't give you a quote."
		},
		"vehicle_too_old": {
			"message": "We can only support vehicles up to {{max}} years old."
		},
		...more
	}
}
```

The values of the `message` field are Mustache templates. They should be compiled with Mustache and executed against the Cuvva Error meta data before being rendered.

### `get_promo_requirement_codes`

Provides messages for the requirements which can be returned when redeeming a promo code.

#### Response

```js
{
	"codes": {
		"no_previous_policies": {
			"message": "This code is valid for your first policy only."
		},
		...more
	}
}
```

### `get_abi_codes`

Provides user-friendly mappings for all ABI codes used by the system.

#### Response

```js
{
	"employment_codes": {
		"E": {
			"name": "Employed",
			"requires_occupation": true
		},
		"F": {
			"name": "Student",
			"requires_occupation": false
		},
		...more
	},
	"occupation_codes": {
		"001": {
			"name": "Abattoir Worker"
		},
		...more
	},
	"location_codes": {
		"1": {
			"name": "Locked garage",
			"sentence_form": "in a locked garage"
		},
		...more
	},
	"relation_codes": {
		"P": {
			"name": "Policyholder"
		},
		...more
	},
}
```

### `get_cue_codes`

Provides user-friendly mappings for all CUE codes used by the system.

```js
{
	"role_codes": {
		"PH": {
			"name": "Policyholder"
		},
		...more
	},
	"ncd_flag_codes": {
		"D": {
			"name": "Disallowed"
		},
		...more
	}
}
```
