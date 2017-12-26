# upload service

Base URLs:

- prod: https://service-upload.prod.ext.cuvva.co
- sandbox: https://service-upload.sandbox.ext.cuvva.co

## API

Conforms to the [Cuvva services standard][1].

### `POST /1/files`

Uploads a new file, records it in the file registry, and returns details about
the file.

For non-internal requests, the `userId` specified in the request body must match
the access token JWT subject.

Unusually, this method does not accept JSON. Instead, you must provide the data
as a standard multipart form, with the relevant content type header.

The `purpose` field is required and must be set to one of: `identity_document`,
`verification_selfie`, and `vehicle_photo`. The `relatedIdentifier` field is
optional.

Example raw request:

```http
POST /1/files HTTP/1.1
Authorization: Bearer 01.eyJhbGciOiJub25lIn0.eyJzdWIiOiI4YmZjYmZmOC00YTFlLTQ4OWEtODFkMS0yZmIxNDFlMTkxNTkifQ.
Host: service-upload.prod.ext.cuvva.co
Content-Type: multipart/form-data; boundary=RandomString123
Content-Length: 437

--RandomString123
Content-Disposition: form-data; name="userId"

8bfcbff8-4a1e-489a-81d1-2fb141e19159
--RandomString123
Content-Disposition: form-data; name="purpose"

vehicle_photo
--RandomString123
Content-Disposition: form-data; name="relatedIdentifier"

56150f6fd0acdc14008e9551
--RandomString123
Content-Disposition: form-data; name="file"; filename="blah.txt"
Content-Type: text/plain

Testing!
--RandomString123--
```

Example response body:

```json
{
	"id": "56c49f985ad3e717007789df",
	"createdAt": "2016-02-17T16:28:08.447Z",
	"userId": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"purpose": "vehicle_photo",
	"relatedIdentifier": "56150f6fd0acdc14008e9551",
	"mime": "text/plain"
}
```

[1]: https://github.com/cuvva/standards/blob/master/services.md
