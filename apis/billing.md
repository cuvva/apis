# billing service

Base URLs:

- dev: https://api.dev.cuv-nonprod.app/1/service-billing
- prod: https://api.prod.cuv-prod.app/1/service-billing
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-billing

Stripe keys:

- dev/sandbox: `pk_test_aECH9zVgwnkPbAmBjWyLdkrr`
- live: `pk_live_eHTSOIlGyXzbHFWtO4fEi9sr`

## API

Conforms to the [Cuvva services standard][1].

### `GET /2/users/:user_id`

Retrieves information about the user.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example response body:

```json
{
	"defaultSourceId": "card_18WOu8FHyMT1SCUFZThyPQrJ",
	"sources": [
		{
			"id": "card_18WOu8FHyMT1SCUFZThyPQrJ",
			"type": "card",
			"finalDigits": "1234",
			"brand": "Visa",
			"expiryMonth": 6,
			"expiryYear": 2015,
			"address": {
				"lines": [
					"3 Lady Lawson St"
				],
				"locality": "Edinburgh",
				"postalCode": "EH3 9DR",
				"countryCode": "GB"
			}
		}
	]
}
```

At present, Stripe IDs are returned directly. This will likely change in the
future. Do not store the IDs for an extended period of time, or rely on them
remaining constant.

### `PUT /2/users/:user_id/default_source`

Sets the user's default source.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject. The `sourceId` must belong to the `user_id`.

Example request body:

```json
{
	"sourceId": "card_18WOu8FHyMT1SCUFZThyPQrJ"
}
```

Response will be a 204 (indicating no content). In the future, this may change
to a 200 with a JSON body without notice.

### `POST /2/users/:user_id/sources`

Adds the token to the user's sources.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example request body:

```json
{
	"stripeToken": "tok_16oI0hFHyMT1SCUFxUHFoUnK",
	"ravelinToken": "tk-8c46447a-fdce-4e48-88aa-234a3c014330"
}
```

Example response body:

```json
{
	"id": "card_18WOu8FHyMT1SCUFZThyPQrJ",
	"type": "card",
	"finalDigits": "1234",
	"brand": "Visa",
	"expiryMonth": 6,
	"expiryYear": 2015,
	"address": {
		"lines": [
			"3 Lady Lawson St"
		],
		"locality": "Edinburgh",
		"postalCode": "EH3 9DR",
		"countryCode": "GB"
	}
}
```

At present, Stripe IDs are returned directly. This will likely change in the
future. Do not store the IDs for an extended period of time, or rely on them
remaining constant.

### `DELETE /2/users/:user_id/sources/:source_id`

Deletes the source from the user.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject. The `source_id` must belong to the `user_id`.

Response will be a 204 (indicating no content). In the future, this may change
to a 200 with a JSON body without notice.

### `POST /2/payments`

Create a payment authorization for a payable.

Handler is idempotent - if a payment already exists for the payable, the
parameters will be checked to ensure they match, then the original response will
be returned. The source information is not checked, as this is not currently
possible.

`payableId`, `userId` and `amount` must be provided. Otherwise, an error will be
returned. `amount` and `userId` must also match their corresponding properties
in the payable.

The source must be specified one of the following three ways:

- `sourceId` - for choosing which source should be charged
- `sourceId` and `threeDSecureToken` - after completing a 3DS challenge
- `stripeToken` - for one-time-use payment methods, such as Apple Pay

If the server determines that 3D Secure is required, it will throw
`threedsecure_verification` with the `threeDSecureToken` and `verificationUrl`
fields in the meta object. The URL should be presented to the user, who can
verify the transaction.

For non-internal requests, `userId` must match the access token JWT subject.

Example request bodies:

```json
{
	"payableId": "8e4b172064ac94c27814fb15",
	"userId": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"sourceId": "card_18WOu8FHyMT1SCUFZThyPQrJ",
	"amount": 500
}
```

```json
{
	"payableId": "8e4b172064ac94c27814fb15",
	"userId": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"sourceId": "card_18WOu8FHyMT1SCUFZThyPQrJ",
	"threeDSecureToken": "tdsrc_8ouyAU4BztyC3b",
	"amount": 500
}
```

```json
{
	"payableId": "8e4b172064ac94c27814fb15",
	"userId": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"stripeToken": "tok_16oI0hFHyMT1SCUFxUHFoUnK",
	"amount": 500
}
```

Example response body:

```json
{
	"id": "471f995d2ecf8865249c200a"
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
