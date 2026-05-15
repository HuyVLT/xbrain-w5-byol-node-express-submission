# Notes

## Strategy

Used `serverless-http` as the Lambda adapter.

## Why this choice

- Keeps `app.js` unchanged and framework-pure.
- Requires only one new entrypoint file plus one dependency.
- Fits the existing split between local dev (`server.js`) and app logic (`app.js`).

## Cold start

- Measured cold start: pending AWS deployment and CloudWatch `Init Duration` capture.