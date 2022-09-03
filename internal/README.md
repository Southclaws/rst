# internal

This is where your infrastructure goes. Anything in the app can import from this code but it's not public due to Go's rules around package paths with `internal` in them.

By "infrastructure" I mean stuff that your application requires to function but is not related to your business logic. For example:

- database clients
- SDKs for external services
- email clients
- just... clients tbh

