---
name: weavr-api
description: Answers questions about the Weavr API, including managed accounts, cards, transactions, and authentication. Use whenever a developer asks how to integrate with Weavr, make API calls, handle errors, or understand request/response formats.
curated: true
version: 1.1.0
---

# Weavr API

API reference: https://docs.weavr.io/api/
OpenAPI: https://docs.weavr.io/oas/multi.json

Use the endpoint schema for required fields and operation-specific states; they vary by resource.

The Weavr API is an embedded finance platform for issuing accounts, cards, and processing payments. It supports corporate and consumer identities with full KYC/KYB verification.

## Environments

| Environment | Base URL                   |
| ----------- | -------------------------- |
| Sandbox     | `https://sandbox.weavr.io` |
| Production  | `https://secure.weavr.io`     |

## Authentication

Multi API calls use these headers:

```
api-key: {{api_key}}              # Server-side API key (all requests)
Authorization: Bearer {{token}}   # End-user token (identity-scoped access token)
Content-Type: application/json
```

The API base path is `/multi`. The UI key is public and initializes SDKs; keep the API key on your server. `programme-key` is the simulator header, not an extra header for Multi identity creation.

Login can return `tokenType: ACCESS` (use it directly) or `AUTH` (call `GET /multi/identities`, then `POST /multi/access_token` with the selected identity). Always handle both. Tokens expire after five minutes of inactivity, with no refresh endpoint. Switching identities exchanges the current Access Token and invalidates it; update every tab that shares the session.

## Quick start

```typescript
// 1. Create a corporate identity
// POST /multi/corporates
// Headers: api-key, Content-Type: application/json
const corporate = await fetch("https://sandbox.weavr.io/multi/corporates", {
  method: "POST",
  headers: {
    "api-key": API_KEY,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    profileId: CORPORATE_PROFILE_ID,
    ipAddress: CUSTOMER_IP_ADDRESS,
    rootUser: {
      name: "John",
      surname: "Smith",
      email: "john@acme.com",
      mobile: { countryCode: "+44", number: "7700900123" },
      companyPosition: "DIRECTOR",
      dateOfBirth: { year: 1985, month: 6, day: 15 },
    },
    company: {
      type: "LLC",
      name: "Acme Ltd",
      registrationNumber: "12345678",
      registrationCountry: "GB",
      businessAddress: {
        addressLine1: "123 High St",
        city: "London",
        postCode: "EC1A 1BB",
        country: "GB",
      },
    },
    baseCurrency: "GBP",
  }),
});

// 2. Set password: POST /multi/passwords/{user_id}/create, where user_id is the root
//    user id from step 1 (via Secure UI tokenization - never handle raw passwords)
// 3. Login: POST /multi/login_with_password → check tokenType: ACCESS (use directly)
//    or AUTH (GET /multi/identities, then POST /multi/access_token with the chosen identity)
// 4. Verify the root user's email and enroll an authentication factor.
//    Optionally pre-populate KYB before starting it (see below).
// 5. Start KYB: POST /multi/corporates/kyb (the corporate comes from the access token)
// 6. After KYB approval, create managed account: POST /multi/managed_accounts
// 7. Issue virtual card: POST /multi/managed_cards
```

## Core endpoints

### Identity management

Standard corporate and consumer endpoints act on the identity selected by the Access Token. KYB pre-population is an exception: it takes a corporate ID and only an API key.

```
POST   /multi/corporates                           Register a corporate
GET    /multi/corporates                           Get the corporate
PATCH  /multi/corporates                           Update the corporate
POST   /multi/corporates/kyb                       Start KYB verification
GET    /multi/corporates/kyb                       Get KYB status

POST   /multi/consumers                            Register a consumer
GET    /multi/consumers                            Get the consumer
PATCH  /multi/consumers                            Update the consumer
POST   /multi/consumers/kyc                        Start KYC verification
GET    /multi/consumers/kyc                        Get KYC status

POST   /multi/users                                Register an authorized user
GET    /multi/users                                List users
GET    /multi/users/{user_id}                      Get a user
PATCH  /multi/users/{user_id}                      Update a user
POST   /multi/users/{user_id}/invite               Invite a user to set a password
POST   /multi/users/{user_id}/activate             Reactivate a user
POST   /multi/users/{user_id}/deactivate           Deactivate a user
```

### Authentication

Password endpoints are all `POST`. Create is the only one that names a user; update and validate act on the logged-in user.

```
POST   /multi/passwords/{user_id}/create           Create a password (first-time setup)
POST   /multi/passwords/update                     Update the logged-in user's password
POST   /multi/passwords/validate                   Check a password against the complexity rules
POST   /multi/passwords/lost_password/start        Start password recovery (sends an email)
POST   /multi/passwords/lost_password/resume       Complete password recovery
POST   /multi/login_with_password                  Log in with a password
POST   /multi/login_via_biometrics                 Log in with biometrics
POST   /multi/access_token                         Exchange an AUTH token for an ACCESS token
POST   /multi/logout                               Log out
GET    /multi/authentication_factors               List authentication factors
POST   /multi/authentication_factors/otp/{channel}        Enrol an OTP factor (step 1, sends the code)
POST   /multi/authentication_factors/otp/{channel}/verify Enrol an OTP factor (step 2, confirms the code)
POST   /multi/authentication_factors/push/{channel}       Enrol a push factor
POST   /multi/stepup/challenges/otp/{channel}             Request step-up (OTP)
POST   /multi/stepup/challenges/otp/{channel}/verify      Verify step-up (OTP)
POST   /multi/stepup/challenges/push/{channel}            Request step-up (push)
```

### Financial instruments

```
POST   /multi/managed_accounts                     Create managed account
GET    /multi/managed_accounts                     List accounts
GET    /multi/managed_accounts/{id}                Get account
GET    /multi/managed_accounts/{id}/statement      Get statement
POST   /multi/managed_accounts/{id}/iban           Assign IBAN to account
GET    /multi/managed_accounts/{id}/iban           Get IBAN details

POST   /multi/managed_cards                        Create a card (prepaid or debit body)
GET    /multi/managed_cards                        List cards
GET    /multi/managed_cards/{id}                   Get a card
PATCH  /multi/managed_cards/{id}                   Update a card
POST   /multi/managed_cards/{id}/block             Block a card
POST   /multi/managed_cards/{id}/unblock           Unblock a card
POST   /multi/managed_cards/{id}/remove            Remove a card
GET    /multi/managed_cards/{id}/statement         Get a card statement
POST   /multi/managed_cards/{id}/physical/activate Activate a physical card
```

### Transfers and payments

```
POST   /multi/transfers                            Internal transfer (account to account)
GET    /multi/transfers/{id}                       Get transfer
POST   /multi/sends                                External send
GET    /multi/sends/{id}                           Get send
POST   /multi/outgoing_wire_transfers              Wire transfer (SEPA/Faster Payments)
GET    /multi/outgoing_wire_transfers/{id}         Get wire transfer
```

### Beneficiaries

```
POST   /multi/beneficiaries                        Add payees (batch, needs step-up)
GET    /multi/beneficiaries                        List payees
GET    /multi/beneficiaries/{id}                   Get a payee
POST   /multi/beneficiaries/remove                 Remove payees (batch)
GET    /multi/beneficiaries/batch/{id}             Get a payee batch
```

## Common request bodies

Replace sample IDs with the appropriate profile and instrument IDs from your programme.

### Create managed account

```json
{
  "profileId": "123456789012345678",
  "friendlyName": "Main Account",
  "currency": "GBP"
}
```

### Create managed card

`POST /multi/managed_cards` has distinct prepaid and debit request schemas in the API reference (`prepaidManagedCardCreate` and `debitManagedCardCreate`). Select the intended mode and its required fields instead of combining examples from the two. Corporate card assignees must be authorized users.

### Create transfer

```json
{
  "profileId": "123456789012345678",
  "source": { "type": "managed_accounts", "id": "123456789012345679" },
  "destination": { "type": "managed_accounts", "id": "123456789012345680" },
  "destinationAmount": { "currency": "GBP", "amount": 1000 }
}
```

## KYB pre-population

Before starting KYB, optionally send information already held during onboarding:

```text
POST /multi/corporates/{corporateId}/kyb/entity      Company and ownership graph
POST /multi/corporates/{corporateId}/kyb/profile     Account-usage profile
POST /multi/corporates/{corporateId}/kyb/documents   Supporting document upload
GET  /multi/corporates/{corporateId}/kyb/entity      Read recorded entity data
GET  /multi/corporates/{corporateId}/kyb/profile     Read recorded profile
GET  /multi/corporates/{corporateId}/kyb/documents   List document metadata
```

These server calls require `api-key`, no user token, and a corporate in your programme. Support must enable pre-population on the profile, including entity submission. Otherwise calls return `NOT_SUPPORTED`.

- Keep company/person `ref` values stable across submissions. Relations point from the holder (`fromRef`) to the company (`toRef`); omit `toRef` to point at the corporate itself.
- Relations can refer to entries in the current or earlier submissions. An `INVALID_REF` failure may leave some entities stored; retry with the same refs after fixing the graph.
- Only the root user can be `isSignatory`. Include their email to identify them; otherwise the call can fail with `SIGNATORY_MUST_BE_ROOT`.
- Wrap account-usage answers in `profile`. Repeat calls merge fields; omitted or empty fields don't clear existing answers. These APIs cannot delete data.
- Upload one document per `multipart/form-data` call: required `file` and `documentType`, optional `description` (500 characters maximum). Files can be up to 10 MB, in PDF, PNG, JPEG, WebP, or DOCX format. Re-uploading stores another copy.
- The root user's email must be verified before Start KYB. On managed onboarding, wait for `COLLECTION.READY_TO_START`; `MANAGED_ONBOARDING_PENDING` leaves pre-population open.
- A successful Start KYB closes the window: the pre-population endpoints return `KYB_ALREADY_STARTED`. The root user reviews and submits through the web KYB component. Uploaded documents go to compliance review and aren't displayed directly in the component.

Track financial access with `corporateKybStatus`, and verification progress with the latest `kybRuns` entry. Render collection or resubmission UI for `COLLECTION.NOT_STARTED`, `COLLECTION.IN_PROGRESS`, or `RESUBMISSION.IN_PROGRESS`; show a waiting state for `REVIEW.IN_PROGRESS`. `kybStatus` and `ongoingKybStatus` are deprecated.

Full flow: https://docs.weavr.io/identities/corporate/kyb-prepopulate/

## Verification of Payee and bulk payments

EUR SEPA outgoing wire transfers normally require showing the VoP result and calling `POST /multi/outgoing_wire_transfers/{id}/confirm`, including transfers to linked accounts. Results include `MATCH`, `CLOSE_MATCH`, `NO_MATCH`, and `NOT_POSSIBLE`.

Corporates can opt out for bulks of at least two OWTs through `verificationOfPayee.bulkPayments` in `GET`/`PATCH /multi/corporates/settings`. The default is `OPTED_IN`. Setting `OPTED_OUT` requires step-up and a certified screen that explains the risks. Without certification, expect `409 VOP_BULK_OPT_OUT_NOT_AVAILABLE`; without step-up, `403 STEP_UP_REQUIRED`. Opting back in doesn't require step-up.

Qualifying opted-out payments return `SEPAVerificationOfPayee: OPTED_OUT` and don't wait in `PENDING_CONFIRMATION`. Single payments remain subject to VoP. Subscribe to the Corporate settings updated webhook to keep the setting current.

Details: https://docs.weavr.io/transactions/owt/vop/overview/

## Card lifecycle and authorization forwarding

- Authorization forwarding may include `posData` and `acquirerData`; every field is optional.
- Destroyed prepaid single-spend cards return leftover funds to the funding account; the adjustment webhook uses `DESTROYED_CARD_BALANCE_TRANSFER`.
- Replacing a `NO_RENEW` card keeps its original expiry. If expiry is less than a month away, replacement fails with `409 INSTRUMENT_EXPIRY_DATE_TOO_CLOSE`.
- Reporting an already-lost card returns `409 INSTRUMENT_ALREADY_MARKED_LOST`; a destroyed card returns `409 INSTRUMENT_DESTROYED`.
- Changing a corporate user's mobile number deactivates their authentication factors; enroll them again.

## Response codes

| Code | Description                          |
| ---- | ------------------------------------ |
| 200  | Success                              |
| 201  | Created                              |
| 400  | Bad request (validation error)       |
| 401  | Unauthorized (invalid/missing auth)  |
| 403  | Forbidden (insufficient permissions) |
| 404  | Not found                            |
| 409  | Conflict (duplicate resource)        |
| 429  | Rate limited                         |

## Common pitfalls

- **Password handling**: Never collect passwords directly. Use Weavr Secure UI components to tokenize sensitive data client-side.
- **KYC/KYB required**: Identities must pass verification before financial instruments can be used.
- **Step-up auth**: Card display and PIN operations require additional step-up authentication.
- **IBAN assignment**: Accounts must be `ACTIVE` before an IBAN can be assigned.
- **Amount format**: Monetary amounts are in minor units (e.g., `1000` = 10.00 GBP).
