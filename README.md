# Data Manager API Conversions by Addingwell

A Google Tag Manager **Server-Side** tag template to send your conversion events (online and offline) to **Google Ads** and **Google Analytics 4** through Google's [Data Manager API](https://developers.google.com/data-manager/api) — the successor of the Google Ads Offline API and the GA4 Measurement Protocol API.

![Overview: the website's GA4 hits feed the Firestore Visitor Context Writer, which stores one document per visitor in Firestore; when a conversion arrives (online or offline), the Reader variable adds that document to the event as aw_visitor_context, and the tag sends the conversion to Google Ads and GA4 in a single API call](./overview.svg)

Paired with the **Firestore Visitor Context** templates (Writer tag + Reader variable), the tag can complete a conversion with what was captured earlier on the website (ad click IDs, session attributes, landing device, user data, client ID), even for an offline conversion sent days later from a CRM. Without them, the tag works on the event data alone.

## Features

- **Server-to-server tracking**: bypass ad blockers, cookie restrictions, and Safari ITP.
- **Offline conversion import** from your CRM, POS, or CDP.
- **Multiple destinations in one request**: Google Ads conversion actions and GA4 properties.
- **Automatic SHA-256 hashing** of user data (already-hashed values are passed through).
- **Consent Mode support**: consent read from the incoming event and normalized (`granted`/`denied`/`true`/`false` → `CONSENT_GRANTED`/`CONSENT_DENIED`).
- **Visitor context fallback**: when present, `eventData.aw_visitor_context` fills in click IDs (gclid/gbraid/wbraid/dclid), consent, client ID, user ID (`identity.user_id`), hashed user data, landing page info and UTMs (client ID: the most recent of `identity.client_ids`) — always with the lowest priority (tag fields > event data > visitor context). The landing page URL/referrer are never taken from `page_location`/`page_referrer` (they describe the current page); they are omitted when no landing value is available. The landing page device info comes from `session_attributes.landing_page_user_agent` and `aw_visitor_context.ad_click.google_ads.landing_device` (`user_agent`, `ip_address`), never from the current request's `user_agent`/`ip_override`.
- **Session attributes**: an encoded session attributes string in `eventData.session_attributes` is sent as-is in `adIdentifiers.sessionAttributes` (Google's recommended method). Otherwise the individual fields (`gad_source`, `gad_campaignid`, `session_start_time_usec`, `landing_page_url`, `landing_page_referrer`) go to `experimentalFields` and the user agent to `landingPageDeviceInfo` (the alternative method); the two are never sent together.
- **Test mode** (`Validate Only`): validate requests without recording conversions.

## Quick start

1. Authorize your service account on the destination: **Standard** role in Google Ads and/or **Editor** role in GA4.
2. Download [`template.tpl`](https://github.com/addingwell/data-manager-api-conversions-tag/blob/master/template.tpl) and import it in your GTM Server container (**Templates → Tag templates → New → ⋮ → Import**).
3. Create a tag from the template, fill in your destinations (Google Ads Customer/Conversion ID, GA4 Property/Measurement ID), and trigger it on your conversion event.
4. Send your events to the GTM Server endpoint in Measurement Protocol (GA4) format:

```json
{
    "events": [{
        "name": "purchase",
        "params": {
            "gclid": "123",
            "transaction_id": "12345",
            "value": "30",
            "currency": "EUR",
            "consent": { "ad_user_data": "CONSENT_GRANTED", "ad_personalization": "CONSENT_GRANTED" },
            "user_data": { "email_address": "john.doe@acme.com", "phone_number": "+33627362122" }
        }
    }]
}
```

5. Test with `Validate Only` set to `true`, then switch it to `false` in production.

## Documentation

Full step-by-step guides (authentication, Measurement Protocol client, field mapping, testing with Postman):

- 🇬🇧 [Offline Google Ads & GA4 conversions with Data Manager API](https://docs.addingwell.com/en/data-manager-api/conversions)
- 🇫🇷 [Conversions offline Google Ads & GA4 avec Data Manager API](https://docs.addingwell.com/fr/data-manager-api/conversions)

## Support

Open an [issue](../../issues) or contact [support@addingwell.com](mailto:support@addingwell.com).

---

Maintained by [Addingwell](https://www.addingwell.com).
