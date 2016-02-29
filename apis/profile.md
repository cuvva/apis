# profile service

Base URLs:

- dev: https://profile-dev.env.cuvva.co
- prod: https://profile-prod.env.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `GET /1/users/:user_id`

Returns the latest profile for the user.

For non-internal requests, `id` must match the access token JWT subject.

Example response body:

```json
{
	"id": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T00:00:00Z",
	"personalName": "James Edward",
	"familyName": "Billingham",
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
	"emailAddress": "james@cuvva.co",
	[...]
}
```

Example response body:

```json
{
	"id": "4b0f9115-f736-4ac8-a8e3-127991dea6cf",
	"createdAt": "2015-01-01T00:00:00Z",
	"updatedAt": "2015-01-01T12:00:00Z",
	"personalName": "James Edward",
	"familyName": "Billingham",
	"emailAddress": "james@cuvva.co",
	[...]
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
