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

## 🌐 Web Resources & Aesthetic Symbols Index
- [SYM 2640](https://sleek-bio-symbols-40.pages.dev/symbol/sym-2640/)
- [HEARTS](https://neon-glitch-symbols-84.pages.dev/ru/hearts/)
- [SYM 1D48E](https://cyberpunk-clan-tags-43.pages.dev/symbol/sym-1d48e/)
- [ROBLOX NAMES](https://cyberpunk-clan-tags-43.pages.dev/es/roblox-names/)
- [SYM 26EF](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-26ef/)
- [DISCORD STATUS](https://cyber-clan-tags-23.pages.dev/ja/discord-status/)
- [SYM 1D489](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-1d489/)
- [SYM 1D407](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d407/)
- [SYM 1D4A1](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-1d4a1/)
- [SYM 26D7](https://vintage-angel-symbols-66.pages.dev/symbol/sym-26d7/)
- [SYM 268A](https://vintage-angel-symbols-66.pages.dev/symbol/sym-268a/)
- [SYM 2732](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-2732/)
- [SYM 2680](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-2680/)
- [LITTLE CAT PAWS KAOMOJI](https://nordic-minimal-fonts-67.pages.dev/symbol/little-cat-paws-kaomoji/)
- [TRENDING](https://vintage-angel-symbols-66.pages.dev/ja/trending/)
- [SYM 1D470](https://theeduplaycampen.pages.dev/symbol/sym-1d470/)
- [SYM 1F600](https://vintage-angel-symbols-66.pages.dev/symbol/sym-1f600/)
- [SYM 1D463](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d463/)
- [SYM 1F60D](https://vintage-angel-symbols-66.pages.dev/symbol/sym-1f60d/)
- [ANGEL WINGS HEART](https://nordic-minimal-fonts-67.pages.dev/symbol/angel-wings-heart/)
- [SYM 1D43B](https://theeduplaycampen.pages.dev/symbol/sym-1d43b/)
- [SYM 2687](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-2687/)
- [SYM 1D478](https://ribbon-heart-fonts-86.pages.dev/symbol/sym-1d478/)
- [SYM 1D43C](https://theeduplaycampen.pages.dev/symbol/sym-1d43c/)
- [SYM 26D1](https://vintage-angel-symbols-66.pages.dev/symbol/sym-26d1/)
- [SYM 26BE](https://vintage-angel-symbols-66.pages.dev/symbol/sym-26be/)
- [GAMING WEAPONS](https://anime-sparkle-text-22.pages.dev/pt/gaming-weapons/)
- [SUPER SHY BLUSHING KAOMOJI](https://angelic-bow-symbols-42.pages.dev/symbol/super-shy-blushing-kaomoji/)
- [OUTLINED STAR](https://coquette-aesthetic-symbols-86.pages.dev/symbol/outlined-star/)
- [SYM 2667](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-2667/)
- [SYM 2683](https://vintage-angel-symbols-66.pages.dev/symbol/sym-2683/)
- [SYM 1D408](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-1d408/)
- [SYM 1D402](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-1d402/)
- [SYM 26E8](https://pearl-girly-fonts-86.pages.dev/symbol/sym-26e8/)
- [SYM 1D430](https://clean-dot-aesthetic-48.pages.dev/symbol/sym-1d430/)
- [SYM 1D404](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d404/)
- [RIGHT HEAVY BRACKET BOX](https://nordic-minimal-fonts-67.pages.dev/symbol/right-heavy-bracket-box/)
- [SYM 26DA](https://coquette-aesthetic-symbols-86.pages.dev/symbol/sym-26da/)
- [SYM 1D40F](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d40f/)
- [SYM 273B](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-273b/)
- [SYM 1D431](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-1d431/)
- [SYM 1D431](https://clean-dot-aesthetic-48.pages.dev/symbol/sym-1d431/)
- [SYM 1D401](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d401/)
- [SYM 1D499](https://cyberpunk-clan-tags-43.pages.dev/symbol/sym-1d499/)
- [SYM 263A FE0F](https://vintage-angel-symbols-66.pages.dev/symbol/sym-263a-fe0f/)
- [TIBETAN LOTUS BLOSSOM](https://mecha-synth-kaomoji-92.pages.dev/symbol/tibetan-lotus-blossom/)
- [SPARKLE DOT FLARE](https://nordic-minimal-fonts-67.pages.dev/symbol/sparkle-dot-flare/)
- [SYM 260A](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-260a/)
- [SYM 1D43C](https://anime-sparkle-text-22.pages.dev/symbol/sym-1d43c/)
- [SPARKLE DOT FLARE](https://coquette-aesthetic-symbols-14.pages.dev/symbol/sparkle-dot-flare/)
- [SYM 2664](https://mecha-synth-kaomoji-92.pages.dev/symbol/sym-2664/)
- [SYM 1F49B](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-1f49b/)
- [SYM 2682](https://vintage-angel-symbols-66.pages.dev/symbol/sym-2682/)
- [SYM 26E1](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-26e1/)
- [SYM 1D48C](https://coquette-symbols.pages.dev/symbol/sym-1d48c/)
- [SYM 1D44C](https://coquette-symbols.pages.dev/symbol/sym-1d44c/)
- [SYM 2676](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-2676/)
- [SYM 1F914](https://anime-sparkle-text-22.pages.dev/symbol/sym-1f914/)
- [SYM 1F49D](https://vintage-angel-symbols-66.pages.dev/symbol/sym-1f49d/)
- [SYM 1F47D](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-1f47d/)
- [SYM 1D47C](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-1d47c/)
- [RIGHTWARDS PAIRED HARPOON](https://nordic-minimal-fonts-67.pages.dev/symbol/rightwards-paired-harpoon/)
- [SYM 2741](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-2741/)
- [SYM 26C1](https://pearl-girly-fonts-86.pages.dev/symbol/sym-26c1/)
- [SYM 1D445](https://clean-dot-aesthetic-48.pages.dev/symbol/sym-1d445/)
- [LEFT RIGHT EXCHANGE ARROWS](https://nordic-minimal-fonts-67.pages.dev/symbol/left-right-exchange-arrows/)
- [SYM 1D427](https://theeduplaycampen.pages.dev/symbol/sym-1d427/)
- [STARRY ELEVATION AURA](https://nordic-minimal-fonts-67.pages.dev/symbol/starry-elevation-aura/)
- [ROBLOX NAMES](https://gothic-bio-fonts-13.pages.dev/vi/roblox-names/)
- [SYM 1D44F](https://theeduplaycampen.pages.dev/symbol/sym-1d44f/)
- [SYM 1D455](https://ribbon-heart-fonts-86.pages.dev/symbol/sym-1d455/)
- [SYM 1F47D](https://anime-sparkle-text-22.pages.dev/symbol/sym-1f47d/)
- [SYM 26FE](https://theeduplaycampen.pages.dev/symbol/sym-26fe/)
- [RIGHT WING CLAN FLARE](https://nordic-minimal-fonts-67.pages.dev/symbol/right-wing-clan-flare/)
- [SYM 1F602](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-1f602/)
- [SYM 1D442](https://theeduplaycampen.pages.dev/symbol/sym-1d442/)
- [SYM 26D0](https://matrix-glitch-text-37.pages.dev/symbol/sym-26d0/)
- [SYM 1F622](https://mecha-synth-kaomoji-92.pages.dev/symbol/sym-1f622/)
- [FREEFIRE NAMES](https://vintage-angel-symbols-66.pages.dev/ja/freefire-names/)
- [SYM 2629](https://anime-sparkle-text-22.pages.dev/symbol/sym-2629/)
- [SIXTEEN POINTED STAR](https://coquette-aesthetic-symbols-86.pages.dev/symbol/sixteen-pointed-star/)
- [RINGED PLANET SATURN](https://coquette-aesthetic-symbols-14.pages.dev/symbol/ringed-planet-saturn/)
- [SYM 26F2](https://matrix-glitch-text-37.pages.dev/symbol/sym-26f2/)
- [STARS](https://gothic-bio-fonts-13.pages.dev/ja/stars/)
- [SYM 1D45D](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-1d45d/)
- [HEARTS](https://coquette-aesthetic-symbols-86.pages.dev/hearts/)
- [SYM 1D44E](https://cyberpunk-clan-tags-43.pages.dev/symbol/sym-1d44e/)
- [SYM 26C0](https://theeduplaycampen.pages.dev/symbol/sym-26c0/)
- [SYM 1F910](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-1f910/)
- [SYM 260D](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-260d/)
- [SYM 26C1](https://theeduplaycampen.pages.dev/symbol/sym-26c1/)
- [BORDERS DIVIDERS](https://pastel-chibi-emotes-23.pages.dev/ru/borders-dividers/)
- [SYM 1D44D](https://cyberpunk-clan-tags-43.pages.dev/symbol/sym-1d44d/)
- [SYM 2749](https://vintage-angel-symbols-66.pages.dev/symbol/sym-2749/)
- [SYM 1D403](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d403/)
- [SYM 26B8](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-26b8/)
- [SYM 1F60F](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-1f60f/)
- [SYM 26DF](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-26df/)
- [FREEFIRE NAMES](https://futuristic-gaming-fonts-52.pages.dev/es/freefire-names/)
- [SYM 1F49A](https://pastel-chibi-emotes-23.pages.dev/symbol/sym-1f49a/)
- [SYM 2654](https://pearl-girly-fonts-86.pages.dev/symbol/sym-2654/)
- [SYM 26F7](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-26f7/)
- [TRENDING](https://anime-sparkle-text-22.pages.dev/ja/trending/)
- [SYM 1F493](https://vintage-angel-symbols-66.pages.dev/symbol/sym-1f493/)
- [SYM 1D40A](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d40a/)
- [BEAMED EIGHTH NOTES](https://coquette-aesthetic-symbols-14.pages.dev/symbol/beamed-eighth-notes/)
- [SYM 267C](https://nordic-minimal-fonts-67.pages.dev/symbol/sym-267c/)
- [SYM 1F979](https://anime-sparkle-text-22.pages.dev/symbol/sym-1f979/)
- [SYM 26E0](https://pearl-girly-fonts-86.pages.dev/symbol/sym-26e0/)
- [BORDERS DIVIDERS](https://coquette-aesthetic-symbols-86.pages.dev/ja/borders-dividers/)
- [FREEFIRE NAMES](https://coquette-aesthetic-symbols-14.pages.dev/es/freefire-names/)
- [SYM 1D417](https://pearl-girly-fonts-86.pages.dev/symbol/sym-1d417/)
- [SYM 267A](https://pearl-girly-fonts-86.pages.dev/symbol/sym-267a/)
- [SYM 1F49F](https://vintage-angel-symbols-66.pages.dev/symbol/sym-1f49f/)
- [SYM 26A9](https://pearl-girly-fonts-86.pages.dev/symbol/sym-26a9/)
- [SYM 2643](https://coquette-aesthetic-symbols-86.pages.dev/symbol/sym-2643/)
- [BRACKETS](https://coquette-aesthetic-symbols-14.pages.dev/ja/brackets/)
- [SYM 2749](https://coquette-aesthetic-symbols-86.pages.dev/symbol/sym-2749/)
- [SYM 26FD](https://kawaii-kaomoji-hub-96.pages.dev/symbol/sym-26fd/)
- [HEAVY HEART EXCLAMATION](https://nordic-minimal-fonts-67.pages.dev/symbol/heavy-heart-exclamation/)
- [SYM 1D4A3](https://matrix-glitch-text-37.pages.dev/symbol/sym-1d4a3/)
- [SYM 2764 FE0F 200D 1FA79](https://vintage-angel-symbols-66.pages.dev/symbol/sym-2764-fe0f-200d-1fa79/)
- [DISCORD STATUS](https://neon-glitch-symbols-84.pages.dev/discord-status/)
- [SYM 2683](https://coquette-aesthetic-symbols-86.pages.dev/symbol/sym-2683/)
- [STARRY ELEVATION AURA](https://futuristic-gaming-fonts-52.pages.dev/symbol/starry-elevation-aura/)
- [SYM 26BB](https://futuristic-gaming-fonts-52.pages.dev/symbol/sym-26bb/)
- [SYM 1D49F](https://ribbon-heart-fonts-86.pages.dev/symbol/sym-1d49f/)
- [SYM 1D45A](https://theeduplaycampen.pages.dev/symbol/sym-1d45a/)
- [SWIMMING FISH LEFT](https://neon-glitch-symbols-84.pages.dev/symbol/swimming-fish-left/)
- [SYM 1D49A](https://cyberpunk-clan-tags-43.pages.dev/symbol/sym-1d49a/)
