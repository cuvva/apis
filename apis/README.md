# APIs

Cuvva's functionality is separated into a number of services, each of which is kept as simple as possible. The APIs of all the public services are documented here.

These services work with the same data in testing and real-life usage. As such, the prod version of these services are shared with dev:

- [auth](auth.md) - provides a simple OAuth implementation, authorizing access to all other services
- [code-mapping](code-mapping.md) - provides human-readable mappings for codelists
- [content](content.md) - provides strings and pages for display to users
- [marketing-consent](marketing-consent.md) - keeps track of users' consent for marketing communications
- [notification](notification.md) - registration of push notification tokens
- [terms](terms.md) - tracks users' acceptance of legal terms
- [upload](upload.md) - handles all binary file uploads (e.g. photos)
- [vehicle](vehicle.md) - allows simple vehicle lookup

This makes it much easier to perform testing day-to-day, as user/vehicle/etc. IDs are the same between dev and prod. For example, a prod file upload can be used with dev too.

As isolated testing of these services is sometimes necessary, we do also maintain a sandbox environment, which is a complete copy of prod.

These services cannot be used between environments:

- [billing](billing.md) - stores users' payment methods and authorizes payments
- [driving-license-registration](driving-license-registration.md) - handles registration of driving licenses for users
- [flexi](flexi.md) - manages the subscription product (formerly known as Flexi)
- [incident](incident.md) - stores users' incidents history
- [intercom](intercom.md) - provides the HMAC signatures needed for Intercom's "secure mode"
- [motor-coverage](motor-coverage.md) - issues quotes and policies for motor insurance
- [news](news.md) - provides news items for display in the apps
- [profile](profile.md) - handles user profile data (name, birth date, etc.)
- [promo](promo.md) - handles our referral scheme, promotions, discounts, etc.
- [user-config](user-config.md) - provides feature flags/config to our apps

By default, accounts only work on prod. If you'd like access to dev or sandbox, drop me an email: james.billingham@cuvva.com
