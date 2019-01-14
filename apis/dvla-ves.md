# service-dvla-ves

Provides vehicle data from the DVLA.

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-dvla-ves/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-dvla-ves/1

## Versions

- 2018-09-25: service created

## Methods

### `get_tax_status`

Retrieves information about the current tax status for a given vehicle.

#### Request

```json
{
	"vehicle_id": "56150f6fd0acdc14008e9551"
}
```

#### Response

```json
{
	"up_to_date": true,
	"due_date": "2018-10-05"
}
```
