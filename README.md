# Amazon Events API Tag — Server-Side Template for Google Tag Manager

This Google Tag Manager server-side template sends conversion events to the Amazon Ads Events API from your GTM server container. It supports automatic GA4 event data integration or fully manual configuration for non-GA4 setups.

## Getting Started

1. In Google Tag Manager, go to **Templates** → **Search Gallery**
2. Search for **Amazon Events API** and add it to your workspace
3. Also add the companion templates: [Amazon CAPI Auth](https://github.com/amzn/ads-capi-gtm-auth) and [Amazon CAPI Timestamp](https://github.com/amzn/ads-capi-gtm-timestamp)
4. Create a variable using the **Amazon CAPI Auth** template — enter your Client ID, Client Secret, and Refresh Token
5. Create a variable using the **Amazon CAPI Timestamp** template (no configuration needed for real-time events)
6. Create a new tag using the **Amazon Events API** template
7. Configure the required fields (see below), set up a trigger, and publish

---

## GA4 Event Schema (Automatic Mode)

In Automatic mode, the tag reads from the GA4 event data flowing through your server container. Your web-side GA4 tag should send events following this schema:

```javascript
gtag('event', 'purchase', {
  transaction_id: '<TRANSACTION_ID>',
  value: 49.99,
  currency: 'USD',
  items: [
    {
      item_id: '<ITEM_SKU>',
      item_brand: '<BRAND>',
      item_category: '<CATEGORY>',
      quantity: 2
    },
    {
      item_id: '<ITEM_SKU_2>',
      item_brand: '<BRAND>',
      item_category: '<CATEGORY>',
      quantity: 1
    }
  ],
  user_data: {
    email_address: '<EMAIL>',
    phone_number: '<PHONE_E164>',
    address: {
      first_name: '<FIRST_NAME>',
      last_name: '<LAST_NAME>',
      street: '<STREET_ADDRESS>',
      city: '<CITY>',
      region: '<STATE>',
      postal_code: '<POSTAL_CODE>'
    }
  }
});
```

| GA4 Field | Maps To |
|---|---|
| `event_name` | Event Name |
| `transaction_id` | Event ID (deduplication) |
| `value` | Event Value |
| `currency` | Currency Code |
| `items[].quantity` | Units Sold (summed across all items) |
| `items[0].item_id` | Product ID |
| `items[0].item_brand` | Brand |
| `items[0].item_category` | Category |
| `user_data.email_address` | Email match key |
| `user_data.phone_number` | Phone match key |
| `user_data.address.first_name` | First Name match key |
| `user_data.address.last_name` | Last Name match key |
| `user_data.address.street` | Address match key |
| `user_data.address.city` | City match key |
| `user_data.address.region` | State match key |
| `user_data.address.postal_code` | Postal Code match key |

Pre-hashed `sha256_email_address` and `sha256_phone_number` keys are also supported.

---

## Configuration

### Configuration Mode

Choose how the tag populates its fields:

- **Automatic** — Auto-reads match keys, event name, value, event ID, and product attributes from GA4 event data flowing through your server container. You only configure Auth, Account ID, Country Code, Conversion Type, Event Source, Event Time, and consent.
- **Manual** — You configure every field yourself using GTM variables or hardcoded values. Use this when your data doesn't come from a GA4 client.

### Required Fields

- **Amazon CAPI Auth Variable** — select the auth variable created above
- **Account ID** — your 18-digit DSP Advertiser ID (found in [Amazon DSP](https://advertising.amazon.com) under Campaign Manager → Advertisers)
- **Country Code** — 2-letter ISO 3166-1 alpha-2 code (US, GB, DE, FR, JP, etc.)
- **Conversion Type** — select the event type (Add to Cart, Lead, Off-Amazon Purchases, Sign Up, etc.)
- **Event Source** — Website, Android, iOS, or Offline
- **Event Time** — use the Amazon CAPI Timestamp variable

---

## Event Name

- **Automatic mode**: Read from the GA4 `event_name` (e.g., `purchase`, `add_to_cart`). You can override this with a custom name.
- **Manual mode**: Set directly with a GTM variable or hardcoded value.

Event names have a 256-character maximum and must not contain special characters.

---

## Match Keys

At least one match key is required for the tag to fire. Match keys enable attribution by linking conversions to ad impressions.

You can send **raw (unhashed) or pre-hashed** data — the template handles both:
- **Unhashed PII**: Automatically normalized (lowercased, trimmed, etc.) and SHA-256 hashed server-side before sending to Amazon.
- **Pre-hashed values**: 64-character hex strings (SHA-256) are detected and passed through without re-hashing. Use GA4 keys like `sha256_email_address` or `sha256_phone_number`.
- **Non-PII identifiers** (MAID, RAMP ID, Match ID, Real ID, Merkle ID, Kantar ID, Full IP Address): Sent as-is, no hashing applied.

**Automatic mode**: Auto-populated from GA4 `user_data` fields (see the mapping table above). You can still add rows to the Match Keys table to supplement with additional types (MAID, RAMP ID, Full IP Address, etc.) or to override a specific auto-detected value.

**Manual mode**: Add rows to the Match Keys table. Select the type and provide the value.

Supported match key types:

| Type | Processing |
|---|---|
| Email | Normalized → SHA-256 hashed |
| Phone | Normalized → SHA-256 hashed |
| First Name / Last Name | Normalized → SHA-256 hashed |
| Address / City / State / Postal Code | Normalized → SHA-256 hashed |
| Mobile Ad ID (MAID) | Sent as-is |
| RAMP ID | Sent as-is |
| Match ID | Sent as-is |
| Real ID (AXM) | Sent as-is |
| Merkle ID | Sent as-is |
| Kantar ID | Sent as-is |
| Full IP Address | Sent as-is |

> **Note:** In Automatic mode, manual rows always take priority. If you add a manual Email row, the auto-detected email from GA4 is ignored.

---

## Event Data

- **Value** — monetary value (max 2 decimal places, e.g., 49.99)
- **Currency Code** — ISO 4217 format (USD, EUR, GBP). Required when Value is set.
- **Units Sold** — positive integer, number of items
- **Event ID** — unique identifier for deduplication

In Automatic mode, these are auto-populated from GA4 event data (see the mapping table above).

> **Note:** Currency Code and Units Sold are restricted to Off-Amazon Purchases per the API spec.

---

## Custom Attributes

Standard attributes:
- **Brand** — product brand
- **Product ID** — product identifier
- **Category** — product category

In Automatic mode, these auto-populate from GA4 event data (see the mapping table above). You can override any of them by setting the field manually.

In Manual mode, set each field directly using a GTM variable or hardcoded value.

Custom Data table (always manually configured in both modes):
- Add rows with a **Name**, **Data Type** (`STRING`, `INTEGER`, or `TIMESTAMP`), and **Value**
- Use this for any additional attributes you want to send to Amazon beyond Brand, Product ID, and Category

---

## Privacy & Consent

Required for EU/UK country codes — the tag will not fire without consent configured. Choose one method:

- **TCF** — IAB TCF v2.2 consent string from your CMP
- **GPP** — Global Privacy Platform string from your CMP
- **Amazon Consent** — direct Amazon consent signals:
  - **Ad Storage** — consent for cookie-based tracking (`GRANTED` or `DENIED`)
  - **User Data** — consent for data processing for advertising (`GRANTED` or `DENIED`)

For EEA/UK traffic, at least one consent signal must be provided. If multiple signals are present, TCF takes precedence.

---

## Deduplication

If you run both the Amazon Ad Tag (client-side) and the Events API (server-side), deduplication prevents double-counting. Set the **Event ID** field to a stable identifier (e.g., transaction ID or order ID) shared between both tags.

---

## Security

All API calls, PII hashing, and token handling occur entirely within your GTM server container. No credentials or raw user data are exposed to client-side code.

See CONTRIBUTING for more information.

## License

Apache 2.0 — see [LICENSE](LICENSE)
