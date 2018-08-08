# service-news

Base URLs:

- dev: https://api.dev.cuv-nonprod.app/1/service-news/1
- prod: https://api.prod.cuv-prod.app/1/service-news/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-news/1

## Versions

- 2018-02-01: service created

## Methods

### `get_news`

Provides an array of current news items.

#### Request

```json
{
	"user_id": "8bfcbff8-4a1e-489a-81d1-2fb141e19159",
	"client": {
		"platform": "ios",
		"version": "1.2.3",
		"build": "1234"
	}
}
```

#### Response

```js
{
	"forced_item_id": "newsitem_66qvqQKJGvmGwo6hjpoG9ubMBv4Wtr98k76yqcMJmmom",
	"items": [
		{
			"id": "newsitem_45gbpjxZLbwPpLYXy5AoR2AE4gDqqxcxanLGgYtshE12",
			"title": "Earn £10 💸",
			"subtitle": "by referring a friend to Cuvva",
			"action": {
				"label": "Refer",
				"type": "show_referral_scheme",
				"params": {}
			},
			"fallback_action": {
				"label": "Refer",
				"type": "web_external",
				"params": {
					"url": "https://cuvva.com"
				}
			}
		},
		{
			"id": "newsitem_66qvqQKJGvmGwo6hjpoG9ubMBv4Wtr98k76yqcMJmmom",
			"title": "App outdated 😥",
			"subtitle": "Cuvva may not work properly - please update now",
			"action": {
				"label": "Update",
				"type": "show_app_update",
				"params": {}
			},
			"fallback_action": null
		},
		{
			"id": "newsitem_DBb1JyjUnMxZLaRtr79StDNczsfEvd1pTimZ141qULDL",
			"title": "Blah",
			"subtitle": "Some notice"
		},
		...more
	]
}
```

The `forced_item_id` field will normally be `null`. If it is provided, the app must always show that item, without a dismissal button, even if normally no news would have be shown at all.

The `action` field can be `null` - in which case, no button should be shown.

If the app doesn't understand the action type, or is unable to support the particular action parameters, the `fallback_action` should be used instead.

## Actions

### `alert_message`

```json
{
	"title": "Blockchain insurance",
	"body": "Find out more about our blockchain insurance!\n\nIt is so good.",
	"options": [
		{
			"label": "Visit website",
			"type": "web_external",
			"params": {
				"url": "https://cuvva.blockchain.insurance"
			}
		},
		{
			"label": "Test",
			"type": "alert_message",
			"params": {
				"title": "uwotm8",
				"body": "Such inception",
				"options": [
					{
						"label": "v deep nesting",
						"type": "web_embedded_basic",
						"params": {
							"url": "http://example.com"
						}
					}
				]
			}
		}
	]
}
```

Show the standard Cuvva alert message popup box.

Always provide an additional option to close/dismiss the alert. The `options` field may be empty.

### `web_external`

```json
{
	"url": "https://google.com"
}
```

Launch the URL into an external web browser - e.g. launch Safari.

### `web_embedded_basic`

```json
{
	"url": "http://example.com"
}
```

Show the website as a modal within an in-app web browser - e.g. a SafariViewController.

The implementation must ensure browser chrome is provided (e.g. buttons to dismiss, navigate back/forward, etc.), as the page itself will not provide a way to dismiss the modal.

### `show_referral_scheme`

Show the referral scheme screen as a modal.

This action doesn't have any parameters.

### `show_app_update`

Show the most relevant screen to prompt the user to update.

This action doesn't have any parameters.

### `compose_support_message`

```json
{
	"prefilled_text": "Hi, I'd like to join the user testing group! 👩🏼‍🎨"
}
```

Open the Intercom message composer screen.

The `prefilled_text` field may be `null`.
