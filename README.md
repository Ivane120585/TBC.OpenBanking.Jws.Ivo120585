# TBC.OpenBanking.Jws

.NET libraries for creating and verifying detached JWS signatures on HTTP messages used by Open Banking integrations.

## What this repository provides

- `src/TBC.OpenBanking.Jws` — HTTP message signing, digest calculation, certificate-chain validation, and signature verification.
- `src/TBC.OpenBanking.Jws.Http` — `HttpClient` delegating handler and certificate locator.
- `src/TBC.OpenBanking.Jws.Tests` — cryptographic and audit behavior tests.
- `src/examples` — runnable examples; they require local configuration and are not production deployments.

This repository is a signing/verification SDK. It does **not** implement an Open Banking gateway, OAuth2/OIDC, consent lifecycle, customer authentication, payment execution, partner onboarding, or an audit database. Those belong in an application or service built around the SDK.

## Build and test

The repository pins its SDK in `global.json` (currently .NET SDK 9.0.307).

```sh
dotnet restore TBC.OpenBanking.Jws.slnx
dotnet build TBC.OpenBanking.Jws.slnx --no-restore
dotnet test TBC.OpenBanking.Jws.slnx --no-restore
```

The GitHub Actions CI and release workflows restore, build, and run tests. CI runs on the repository's `Ivane1985` default branch and on `master` during branch transitions. NuGet publishing is tag-triggered and requires the `NUGET_API_KEY` repository secret.

## Audit events

`HttpSigner<T>` and `HttpSignatureVerifier<T>` accept an optional `IAuditSink`. `JwsClientOptions.AuditSink` connects the same sink to outgoing and incoming operations handled by `JwsMessageHandler`.

```csharp
var options = new JwsClientOptions
{
    AuditSink = auditSink, // implement IAuditSink using your approved audit store
};
```

Each event records the operation, outcome, UTC timestamp, algorithm, certificate thumbprint when available, and failure exception type. It deliberately excludes request/response bodies, header values, signatures, and exception messages. The sink is called synchronously; its failures propagate to the caller rather than being silently ignored. Keep the implementation fast, durable, access-controlled, and append-only. Add correlation IDs in the hosting application, where request context is available.

The library does not provide durable storage or claim that a normal application log is a tamper-evident audit trail. Production deployments must configure and monitor their own sink. No audit event is emitted for requests that skip signing or responses that the HTTP handler intentionally skips validating.

## Configuration and examples

Never commit private keys, passwords, tokens, or real customer data. Example configuration must be supplied locally:

- `SampleConsole`: `SAMPLE_CERTIFICATE_PATH` and optionally `SAMPLE_CERTIFICATE_PASSWORD`.
- `ConsoleHttpClientExample`: `JWS_EXAMPLE_BASE_URL` must be an explicit HTTPS URL.
- `WebApiHttpClientExample`: `OpenBankingConnection__BaseUrl`, `OpenBankingConnection__RedirectUri`, `OpenBankingConnection__ClientCertificate`, and `Jws__Options__SigningCertificate`; the endpoint and redirect URI must use HTTPS.

See [SECURITY.md](SECURITY.md) for deployment boundaries, key handling, and reporting guidance.

## Production integration checklist

- Use a private-key provider with appropriate access controls; prefer an HSM or managed key service where the deployment requires it.
- Restrict accepted partner certificates to the trust anchors and identity policy required by the relevant bank/network. The default chain build uses the host platform's certificate trust configuration; embedding `x5c` in a signature does not itself make that certificate trusted.
- Enable revocation checking and signature-time checks where supported by the integration, and monitor their availability and failures.
- Supply a durable `IAuditSink`, protect its records, and alert on audit-storage failures.
- Test certificate rotation, expiry, revocation, partner outages, malformed messages, and replay/idempotency handling in the integrating service.
- Keep OAuth, consent, authorization, rate limiting, business validation, and operational dashboards in the service layer; a valid signature alone does not authorize a transaction.

## License

See [LICENSE.txt](LICENSE.txt).
