# promo service

Base URLs:

- dev: https://service-proxy.dev.ext.cuvva.co/1/service-promo
- prod: https://service-proxy.prod.ext.cuvva.co/1/service-promo
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-promo

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/users/:user_id/referral_info`

Returns the referral information for a given user, including their referral code
and the referral instructions which explain how the code can be used (can be
`null`).

For non-internal requests, `user_id` must match the access token JWT subject.

The code stored internally will be prefixed with `U` to differentiate referral
codes from potential future promo codes.

The existence of the user referred to by `user_id` will not be checked. Referral
codes will be generated for all `user_id`s provided regardless of existence.

Example response body:

```json
{
	"code": "UWKJ4P4VUC",
	"instructions": "We'll give you a pat on the back when someone you refer buys their first policy.",
	"shareText": "If you sign up for Cuvva with my code, we both get a pat on the back. \uD83D\uDE42"
}
```

### `POST /1/users/:user_id/redeem`

Redeem a code for a given user. Currently only supports user referral codes.

For non-internal requests, `user_id` must match the access token JWT subject.

The code provided must begin with `U` (for user). Other codes may become valid
in the future, if support is added for promo codes.

Example request body:

```json
{
	"code": "UWKJ4P4VUC"
}
```

Successful requests will not return a result (although they may in the future).

[1]: https://github.com/cuvva/standards/blob/master/services.md
