---
name: weavr-sdk-web
description: Integrate Weavr Secure UI into browser apps. Covers hosted SDK setup, tokenized credentials, card display and PIN capture, KYC/KYB components, styling, and framework integration. Use when building web Weavr integrations.
curated: true
version: 1.1.0
---

# Weavr Web SDK

Documentation: https://docs.weavr.io/sdks/web/

## Load and initialize

```html
<!-- Sandbox -->
<script src="https://sandbox.weavr.io/app/secure/static/client.1.js"></script>
<!-- Live -->
<script src="https://secure.weavr.io/app/secure/static/client.1.js"></script>
```

Call `window.OpcUxSecureClient.init(uiKey)` once per page load with the public UI key for the selected environment. REST API keys stay on the server.

## Capture credentials

```javascript
const form = window.OpcUxSecureClient.form();
const password = form.input("password", "password", {
  placeholder: "Password",
  strengthCheck: true,
});
const confirmation = form.input("confirmPassword", "confirmPassword", {
  placeholder: "Confirm password",
});
password.mount("#password");
confirmation.mount("#confirm-password");

password.on("passwordValidCheck", ({ status, message }) => {
  // Update validation UI; status is idle, loading, success, or error.
});

function submit() {
  form.tokenize(
    (tokens) => {
      // Send tokens.password to your server for password creation or login.
    },
    (error) => {
      // Show validation feedback.
    }
  );
}
```

The first `form.input` argument names the returned token key. Pair confirmation fields in the same form. Passcode component types are `passCode` and `confirmPassCode`; passcode mode requires support enablement. Updating an existing password or passcode requires step-up. Use `passwordValidCheck` for strength feedback; the older `strength` event is deprecated.

Server login uses `POST /multi/login_with_password`. Inspect `tokenType`: exchange `AUTH` for an identity-scoped Access Token before using financial APIs.

## Display card data

Fetch the card on your server using a stepped-up Access Token. Pass tokenized values to spans, not card IDs:

```javascript
window.OpcUxSecureClient.associate(
  `Bearer ${accessToken}`,
  () => {
    window.OpcUxSecureClient.span("cardNumber", card.cardNumber.value)
      .mount("#card-number");
  },
  (error) => {
    // Handle association failure before mounting.
  }
);
```

Use the matching span type for CVV or PIN according to the [card component guide](https://docs.weavr.io/sdks/web/card-components/). Sensitive response fields may be absent or null without step-up. Mount on reveal and remove the component when hidden; your JavaScript doesn't receive the raw displayed value.

All card components, including PIN capture, require step-up. Web PIN capture currently displays digits while typing and has no built-in confirmation input; use the documented [PIN capture UX](https://docs.weavr.io/sdks/web/card-components#capture-card-pin-behaviour).

## Verification

| Flow | Component | Session setup |
| ---- | --------- | ------------- |
| Consumer KYC | `OpcUxSecureClient.consumer_kyc()` | Start KYC through the API and use its reference. |
| Corporate KYB | `OpcUxSecureClient.kyb()` | Associate the Access Token and use the Start KYB reference. |
| Stakeholder due diligence | `OpcUxSecureClient.kyc()` | Use the emailed reference; no stakeholder login required. |

```javascript
window.OpcUxSecureClient.associate(`Bearer ${accessToken}`, () => {
  window.OpcUxSecureClient.kyb().init(
    "#kyb-container",
    { reference: kybReference },
    (messageType, payload) => {
      // Events include kybSubmitted, kybApproved, kybRejected, and error.
    },
    { lang: "en" }
  );
});
```

Optional [KYB pre-population](https://docs.weavr.io/identities/corporate/kyb-prepopulate/) happens before Start KYB. Use `corporateKybStatus` and the latest `kybRuns` entry to decide whether to render collection/resubmission, show review progress, or handle the outcome. Don't use a generic `complete` event as the source of approval.

## Styling and frameworks

Pass `fonts` at initialization and `style` per component. Input states include `base`, `empty`, `valid`, and `invalid`; focus styling uses the `:focus` pseudo-class. Follow the [React and Vue framework guides](https://docs.weavr.io/sdks/web/frameworks/) for script readiness, mounting, and cleanup, and don't remount secure fields on every render.
