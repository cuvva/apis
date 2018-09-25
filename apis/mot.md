# service-mot

Base URLs:

- prod: https://api.prod.cuv-prod.app/1/service-mot/1
- sandbox: https://api.sandbox.cuv-nonprod.app/1/service-mot/1

## Versions

- 2018-09-24: service created

## Methods

### `get_vehicle_status`

Retrieves information about the current MOT status for a given vehicle.

#### Request

```json
{
	"vehicle_id": "56150f6fd0acdc14008e9551"
}
```

#### Response

```json
{
	"failed_last_test": false,
	"up_to_date": true,
	"expiry_date": "2018-10-05"
}
```
