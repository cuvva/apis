# billing service

Base URLs:

- dev: https://billing-dev.env.cuvva.co
- prod: https://billing-prod.env.cuvva.co

Stripe keys:

- test: `pk_test_aECH9zVgwnkPbAmBjWyLdkrr`
- live: `pk_live_eHTSOIlGyXzbHFWtO4fEi9sr`

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/users/:user_id/default_source`

Retrieves information about the user's default payment source.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example response body:

```json
{
	"finalDigits": "1234",
	"brand": "Visa",
	"expiryMonth": "06",
	"expiryYear": "2015",
	"address": {
		"line1": "3 Lady Lawson St",
		"line2": null,
		"city": "Edinburgh",
		"region": null,
		"postcode": "EH3 9DR",
		"country": "GB"
	}
}
```

### `PUT /1/users/:user_id/default_source`

Sets the user's default payment source.

`stripeToken` must be provided. Otherwise, an error will be returned.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example request body:

```json
{
	"stripeToken": "tok_me2ow5LhVf5kZQ"
}
```

Example response body:

```json
{
	"finalDigits": "4242",
	"brand": "Visa",
	"expiryMonth": "01",
	"expiryYear": "2020",
	"address": {
		"line1": "3 Lady Lawson St",
		"line2": null,
		"city": "Edinburgh",
		"region": null,
		"postcode": "EH3 9DR",
		"country": "GB"
	}
}
```

### `POST /2/payments`

Create a payment authorization for a payable.

Handler is idempotent - if a payment already exists for the payable, the
parameters will be checked to ensure they match, then the original response will
be returned. `stripeToken` is not checked, as this is not currently possible.

`payableId`, `userId` and `amount` must be provided. Otherwise, an error will be
returned.

`amount` and `userId` must match their corresponding properties in the payable.

Optionally, `stripeToken` can be provided to authorize against a one-time-use
payment method, such as Apple Pay.

For non-internal requests, `userId` must match the access token JWT subject.

Example request body:

```json
{
	"payableId": "8e4b172064ac94c27814fb15",
	"userId": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"stripeToken": "tok_dQbrSBf7S5duyA",
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
