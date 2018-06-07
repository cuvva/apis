# service-marketing-consent

Base URLs:

- prod: https://service-proxy.prod.ext.cuvva.co/1/service-marketing-consent/1
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-marketing-consent/1

## Versions

- 2018-05-18: service created

## Methods

### `get_user_status`

Provides information about the user's current decisions.

#### Request

```json
{
	"user_id": "user_000000BQxL1V00QLjcyJrk9qQddse"
}
```

#### Response

```json
{
	"set_at": "2018-05-23T12:19:10Z",
	"email_setting": "relevant"
}
```

The `email_setting` field specifies what kinds of marketing emails we're allowed to send to the user:

- `null`: the user has not made a decision yet (`set_at` will also be `null`)
- `"anything"`: no restrictions - e.g. generic newsletters
- `"relevant"`: only specific/targeted messages - e.g. beta program invitations
- `"essential"`: only important transactional/account emails - no marketing

### `set_user_status`

Updates the user's current decisions.

If emails are set to essential-only, the user will be unsubscribed on Intercom etc. For any other value, they will be re-subscribed (even if unsubscribed by some other method).

#### Request

```json
{
	"user_id": "user_000000BQxL1V00QLjcyJrk9qQddse",
	"email_setting": "essential"
}
```

Values as above.
