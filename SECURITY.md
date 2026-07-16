# Security Policy

## Supported Version

| Version | Supported |
| --- | --- |
| 5.0 | Yes |

## Reporting Security Issues

Do not open a public issue containing secrets, webhook URLs, account identifiers, private exchange data, or exploit details.

Report security-sensitive findings privately to the project owner or repository maintainer.

## Sensitive Data

This repository should not contain:

- TradingView webhook URLs.
- Exchange API keys or secrets.
- Broker credentials.
- Private account IDs.
- Paid data feed credentials.
- Screenshots exposing balances or account information.

## Alert And Webhook Safety

The strategy can emit plain text or JSON alerts. Treat alert payloads as production interfaces:

- Validate downstream webhook receivers.
- Use authentication on external webhook endpoints.
- Do not embed secrets in alert messages.
- Rate-limit automation that consumes TradingView alerts.
- Paper test execution bridges before live deployment.

## Trading Risk

Security reports should distinguish software/security risk from market risk. This project does not guarantee trading performance, fills, uptime, or execution quality.
