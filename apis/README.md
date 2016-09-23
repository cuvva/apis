# APIs

Cuvva's functionality is separated into a number of services, each of which is
kept as simple as possible. The APIs of all the public services are documented
here.

The [auth service](auth.md) provides a simple OAuth/OpenID implementation and
issues access tokens which can be used to access all the other services below.

These services don't do anything of consequence and thus don't need sandboxes
and only run in production:

- [notification](notification.md) - registration of push notification tokens
- [upload](upload.md) - handles all binary file uploads (e.g. photos)
- [vehicle](vehicle.md) - allows simple vehicle lookup

These services have both development and production endpoints to allow for safe testing:

- [billing](billing.md) - stores users' payment methods and authorizes payments
- [garage](garage.md) - tracks ownership of vehicles and suggests relevant vehicles
- [motor coverage](motor-coverage.md) - issues quotes and policies for motor insurance
- [profile](profile.md) - handles user profile data (name, birth date, etc.)
- [promo](promo.md) - tracks user referral (and will also do voucher codes)

The production-only services can be used with the development services also.
i.e. any file upload can be used with both the development and production
systems.

By default, accounts only work on the production systems. If you'd like access
to the development servers, drop me an email at
[james.billingham@cuvva.com](mailto:james.billingham@cuvva.com).
