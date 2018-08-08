# service-terms

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-terms/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-terms/1

## Versions

- 2018-05-18: service created

## Methods

### `get_latest_documents`

Provides the latest available documents.

#### Response

```json
{
	"terms": {
		"version": "2018-05-18",
		"url": "https://example.com"
	},
	"privacy": {
		"version": "2018-05-18",
		"url": "https://example.com"
	}
}
```

### `get_user_status`

Provides information about what agreements currently exist with the user, and whether those agreements are currently up-to-date or not.

#### Request

```json
{
	"user_id": "user_000000BQxYpHDcas3r9ErkyGyYAPQ"
}
```

#### Response

```json
{
	"status": "update_mandatory",
	"terms": {
		"agreed_at": "2018-05-14T23:34:12Z",
		"version": "2018-05-18",
		"url": "https://example.com"
	},
	"privacy": null
}
```

The `status` field may be:

- `up_to_date` - no updates are available
- `update_suggested` - the user should be asked to agree to the latest documents
- `update_mandatory` - most services unusable until agreed to the latest documents

The `terms` and `privacy` fields have the same structure. Either can be `null`, meaning no agreement is in place.

When a user's account is first created, `terms` and `privacy` will be `null` and `status` will be `update_mandatory`.

### `record_agreement`

Records a user's agreement to the terms.

If the specified version isn't the latest, `old_document_version` will be thrown.

#### Request

```json
{
	"user_id": "user_000000BQxYpHDcas3r9ErkyGyYAPQ",
	"document": "terms",
	"version": "2018-05-18"
}
```
