# Webhook Signature Verifier

Verify webhook signatures from Stripe, GitHub, Slack, Twilio, and any HMAC-based service. Runs entirely in your browser.

**Live demo:** https://0xelitesystem.github.io/webhook-signature-verifier/

## Use

Open [`index.html`](./index.html) in a browser. Pick the provider tab. Paste the signature header, secret, and request body. Click Verify.

The tool tells you whether the signature matches and explains the computation.

## Why this exists

When a webhook fails to verify in your application, the question is usually one of:

- Was the body re-serialized somewhere in the path (breaking byte-equality)?
- Is the timestamp tolerance correct?
- Is the secret you're using actually the right secret?
- Is your code computing the signature on the right bytes?

This tool is the reference computation. Compare what your code does against what this tool does, and you'll find the discrepancy fast.

## Supported providers

### Stripe

Verifies the `Stripe-Signature` header. Computes HMAC-SHA256 of `{timestamp}.{body}` with your `whsec_...` secret. Warns if timestamp is older than 5 minutes (Stripe's recommended tolerance).

### GitHub

Verifies the `X-Hub-Signature-256` header. Computes HMAC-SHA256 of the raw body. The legacy `X-Hub-Signature` (SHA-1) is not supported because you shouldn't be using it.

### Slack

Verifies the `X-Slack-Signature` header. Computes HMAC-SHA256 of `v0:{timestamp}:{body}` with your signing secret. Warns if timestamp drift exceeds 5 minutes.

### Twilio

Verifies the `X-Twilio-Signature` header. Computes HMAC-SHA1 (base64-encoded) of the URL plus sorted form parameters, with your auth token.

### Generic HMAC

For services not listed above. Pick the hash (SHA-1, SHA-256, SHA-384, SHA-512), encoding (hex or base64), paste the secret and body, and verify.

## Privacy

All verification uses the browser's Web Crypto API. Secrets never leave your machine. Open DevTools and watch the network tab to confirm, no requests are made.

That said, **don't paste production webhook secrets into any web tool**. Use this for development and debugging.

## Adding a new provider

Open a PR with:

1. Provider documentation link
2. Sample webhook payload + signature you can verify against
3. The signature format (what header, what bytes are signed, what hash)

The verifier code is in `index.html`. Keep it readable; this isn't a place for clever metaprogramming.

## Run locally

```bash
git clone https://github.com/0xelitesystem/webhook-signature-verifier
cd webhook-signature-verifier
# Open index.html
```

## Common debugging hints

If you're seeing "signature does NOT match" but you're sure the secret is right:

1. **Body byte-equality.** Most frameworks reformat JSON bodies (sorting keys, changing whitespace) before they reach your handler. The signature is computed on the original bytes. Capture the raw body BEFORE any framework parses it.
2. **Trailing newline.** Some HTTP servers add or strip trailing newlines.
3. **Encoding.** UTF-8 with BOM vs without BOM produces different signatures.
4. **Wrong secret environment.** Test secret vs live secret. The webhook endpoint and the secret must come from the same Stripe / GitHub / Slack environment.

## Build

There is no build. Single HTML file.

## License

MIT.

## Related

- [jwt-inspector](https://github.com/0xelitesystem/jwt-inspector), decode and verify JWTs
- [webhook-inspector](https://github.com/0xelitesystem/webhook-inspector), inspect webhook payloads
- [oauth-flow-debugger](https://github.com/0xelitesystem/oauth-flow-debugger), decode OAuth URLs
