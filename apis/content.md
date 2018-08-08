# service-content

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-content/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-content/1

## Versions

- 2018-07-23: service created

## Methods

### `get_document_base`

Provides URLs to the base resources needed to render all documents.

When fetching content from the URLs, provide the `ETag` from any existing cached content via the `If-None-Match` header and check for a 304 response.

#### Response

```json
{
	"css_url": "https://example.com",
	"logo_url": "https://example.com"
}
```

### `get_strings`

Returns string mappings for display to customers - e.g. label copy etc.

#### Response

```js
{
	"strings": {
		"travel.inputs.medical.smoker": "Smoked (regularly <u>or</u> socially) for a period of <b>15 years</b> or longer?",
		"travel.inputs.medical.currently_medicated": "Have you taken and/or been prescribed or recommended <span style=\"color: red\">any</span> medication in the last <i>6 months</i>?",
		"travel.inputs.excess": "Excess per incident",
		"travel.inputs.excess.detail": "Maximum you pay in a claim",
		...more
	}
}
```

### `get_pages`

Returns URLs to long-form HTML content for display to customers, along with styling data - e.g. explanations for what is/isn't covered.

When fetching content from the URL, provide the `ETag` from any existing cached content via the `If-None-Match` header and check for a 304 response.

#### Response

```js
{
	"default_css_url": "https://example.com",
	"pages": {
		"legal.driving_license_fon": {
			"css_url": "https://example.com",
			"html_url": "https://example.com"
		},
		"travel.cover_explainers.medical": {
			"html_url": "https://example.com"
		},
		...more
	}
}
```

Where `css_url` is provided within a given page, it is used instead of the default CSS.
