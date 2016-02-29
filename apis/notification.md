# notification service

Base URL:

- https://notification-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `POST /1/users/:user_id/devices`

Registers a device against a user.

The `type` field may be set to `apple`, `amazon` or `google`.

For non-internal requests, `user_id` must match the access token JWT subject.

Example request body:

```json
{
	"type": "apple",
	"token": "00fc13adff785122b4ad28809a3420982341241421348097878e577c991de8f0"
}
```

Response will be a 204 (indicating no content). In the future, this may change
to a 200 with a JSON body without notice.

[1]: https://github.com/cuvva/standards/blob/master/services.md
