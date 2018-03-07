# profile service

Base URLs:

- dev: https://service-proxy.dev.ext.cuvva.co/1/service-profile
- prod: https://service-proxy.prod.ext.cuvva.co/1/service-profile
- sandbox: https://service-proxy.sandbox.ext.cuvva.co/1/service-profile

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/users/:user_id`

Returns the latest profile for the user.

For non-internal requests, `id` must match the access token JWT subject.

Example response body:

```json
{
	"id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T00:00:00Z",
	"firstName": "James",
	"lastName": "Billingham",
	[...]
}
```

### `PUT /1/users/:user_id`

Update the profile, returns the new state of the profile.

For non-internal requests, the `user_id` parameter must match the access token
JWT subject.

Example request body:

```json
{
	"email": "james@cuvva.com",
	[...]
}
```

Example response body:

```json
{
	"id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T12:00:00Z",
	"firstName": "James",
	"lastName": "Billingham",
	"email": "james@cuvva.com",
	[...]
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
