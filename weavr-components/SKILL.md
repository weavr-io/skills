---
name: weavr-components
description: Build Weavr integration flows using Secure UI and the API. Covers tokenized capture and display, corporate and consumer onboarding, KYB pre-population, and choosing web or mobile components. Use when designing a Weavr integration across UI and backend.
curated: true
version: 1.1.0
---

# Weavr component integration

## Security model

Capture components return opaque tokens for passwords, passcodes, OTPs, and card PINs. Forward tokens to your server for the API call; don't collect raw credentials in ordinary inputs.

Display components receive tokenized PAN, CVV, or PIN values fetched by your server. After step-up and SDK association, the SDK renders the value inside its secure view. Pass the tokenized value, not the card ID. Keep display behind a reveal control and remove the component when hidden.

Web card PIN capture currently shows digits during entry and has no built-in confirmation field. Don't assume every secure input masks characters. Follow the [card PIN guidance](https://docs.weavr.io/sdks/web/card-components#capture-card-pin-behaviour).

## Choose an SDK

| Capability | Web | iOS | Android | React Native |
| ---------- | --- | --- | ------- | ------------ |
| Password, OTP, card PIN capture | Yes | Yes | Yes | Yes |
| PAN, CVV, card PIN display | Yes | Yes | Yes | Yes |
| Consumer KYC | Yes | Yes | Yes | Yes |
| Corporate KYB | Yes | No | No | No |
| Passcode authentication | Opt-in | No | Existing apps | Android only |

Prefer passwords for new mobile integrations. Mobile corporate onboarding needs a web handoff for KYB. Use the platform's setup and method signatures rather than translating web APIs into native APIs.

Security model: https://docs.weavr.io/sdks/web/secure-components-model/

## Corporate onboarding

1. Register through `POST /multi/corporates` on your server, supplying the corporate profile ID, root user, company, customer IP address, and base currency.
2. Set the root user's password using Secure UI tokenization and `POST /multi/passwords/{user_id}/create`. Verify their email and enroll an authentication factor.
3. Handle both login outcomes: use `ACCESS` directly, or exchange `AUTH` through `GET /multi/identities` and `POST /multi/access_token`.
4. Optionally pre-populate company/ownership data, the account-usage profile, and documents before Start KYB. These calls use the API key and a corporate ID, without a user token, and require support enablement.
5. Call `POST /multi/corporates/kyb` with the Access Token. In managed onboarding, wait for `COLLECTION.READY_TO_START`. A successful start closes the pre-population window.
6. Associate the Access Token with the Web SDK, then mount `OpcUxSecureClient.kyb().init()` using the returned `reference`.
7. Track the corporate KYB webhook or `GET /multi/corporates/kyb`. Use `corporateKybStatus` for financial access and the latest `kybRuns` status for UI progress. A submission callback doesn't establish approval.

The root user reviews pre-populated information and completes missing details. Uploaded documents go to compliance review. For `RESUBMISSION.IN_PROGRESS`, send the user back to the component; for `REVIEW.IN_PROGRESS`, show a waiting state.

Stakeholder due diligence is a separate `OpcUxSecureClient.kyc()` flow opened from an emailed reference. It doesn't require the stakeholder to authenticate.

- [Corporate root user setup](https://docs.weavr.io/identities/authorised-users/root-user-setup/)
- [KYB pre-population](https://docs.weavr.io/identities/corporate/kyb-prepopulate/)
- [Web KYB components](https://docs.weavr.io/sdks/web/kyb-components/)

## Consumer onboarding

Create the consumer, set up their credentials, and call `POST /multi/consumers/kyc`. On web, mount `OpcUxSecureClient.consumer_kyc()` with the returned reference. On mobile, use the platform's KYC component. Track the API/webhook outcome before enabling financial services.

## Card display

1. Obtain an identity-scoped Access Token and complete step-up.
2. Fetch the card from your backend using `GET /multi/managed_cards/{id}`. Sensitive fields are conditional on step-up; the PAN token is `cardNumber.value`.
3. Associate the stepped-up token with the SDK.
4. Mount the appropriate secure display with its tokenized value when the cardholder requests reveal. Hide it promptly afterward.

Native clipboard copying is opt-in and writes plain text to the system clipboard. Enable it only when the product needs it, after step-up, and explain what was copied.

## Keys and environments

Use the public UI key for SDK initialization and keep the API key on your server. Sandbox and live keys differ. The web bundles are:

```html
<!-- Sandbox -->
<script src="https://sandbox.weavr.io/app/secure/static/client.1.js"></script>
<!-- Live -->
<script src="https://secure.weavr.io/app/secure/static/client.1.js"></script>
```

Initialize `window.OpcUxSecureClient.init(uiKey)` once per page load. For framework mounting and cleanup, follow the [framework guides](https://docs.weavr.io/sdks/web/frameworks/).
