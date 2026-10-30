---
name: weavr-glossary
description: Explain Weavr platform terminology, including programmes and workspaces, profiles, customers and users, managed instruments, trusted payees, verification, and authentication tokens. Use when interpreting Weavr docs or integration concepts.
curated: true
version: 1.1.0
---

# Weavr glossary

Source glossary: https://docs.weavr.io/glossary/

## Platform and ownership

- **Workspace**: your company's Embedder Portal account, containing team members and one or more programmes. Portal roles are workspace-wide.
- **Programme**: a Weavr configuration for your app. Customers, instruments, and transactions belong to a programme; sandbox and live have separate configurations.
- **Profile**: a template for a resource type, such as a corporate, consumer, account, card, transfer, or OWT. Its Profile ID selects configured currencies, limits, countries, branding, and fees.
- **Customer / identity**: a corporate or consumer that owns financial instruments. Identity is the older term still used in API names.
- **Corporate**: a business customer that must complete KYB.
- **Consumer**: an individual customer that must complete KYC.
- **Root user**: the primary user created with a customer. A corporate root user is a legal representative and has the Admin role.
- **Authorized user**: a person invited to act for a customer, without becoming its legal owner. One corporate user can have access to multiple corporate identities with different roles.
- **Card assignee**: the person using a card. A corporate owns its cards; their assignees must be authorized users. For consumers, the owner and assignee are the same person.

## Instruments and payments

- **Managed account**: a stored-value account owned by a customer, with bank details assignable for external transfers. Available currencies depend on the programme/profile.
- **Managed card**: a virtual or physical payment card. Prepaid cards have their own balance; debit cards spend from a linked managed account.
- **Transfer**: movement between Weavr instruments owned by the same customer.
- **Send**: movement to another customer's instrument or to a beneficiary, subject to the configured destination and authentication rules.
- **Outgoing wire transfer (OWT)**: payment from a managed account to an external bank account.
- **Incoming wire transfer (IWT)**: payment from an external bank account into a managed account.
- **Trusted payee / beneficiary**: a saved trusted recipient and its bank or instrument details. Beneficiary remains in API paths. Trust may reduce SCA requirements; it doesn't universally bypass VoP confirmation.
- **Linked account**: an external bank account linked to a customer under the programme's account-linking rules.

## Verification

- **KYC**: personal due diligence for a consumer or a corporate stakeholder.
- **KYB**: business due diligence, including company details, ownership, stakeholders, and account usage. The interactive KYB component is web-only.
- **KYB pre-population**: optional server-side submission of company/ownership data, account-usage answers, and documents before KYB starts. It doesn't replace the root user's review and submission.
- **UBO**: ultimate beneficial owner, the person who ultimately owns or controls a business. Use the onboarding flow's requirements for thresholds and evidence.
- **PEP**: politically exposed person, subject to additional due diligence.
- **VoP**: Verification of Payee for EUR SEPA OWTs. Qualifying corporate bulk payments can be opted out through a certified flow; single payments are always checked.
- **CoP**: Confirmation of Payee for GBP payments. Its linked-account confirmation rules differ from EUR VoP.

## Keys and tokens

- **API key**: secret server credential in the Multi API's `api-key` header. The simulator accepts the API client key under `programme-key` instead.
- **UI key**: public, environment-specific credential for SDK initialization. It isn't a REST API credential.
- **Auth Token (`AUTH`)**: authenticates the user without choosing a customer. Exchange it for an Access Token after selecting an identity.
- **Access Token (`ACCESS`)**: authenticates a user acting for a particular identity and carries that user's roles there. Use it for identity-scoped APIs and SDK association.
- **Step-up**: additional authentication that makes an Access Token eligible for sensitive operations, including secure card display and PIN capture.
- **SCA**: Strong Customer Authentication, used to authorize sensitive financial actions.
- **Tokenized value**: opaque representation of a captured credential or sensitive card field. It isn't interchangeable with a session token or a card ID.
- **Secure UI**: web or native SDK components that capture or display sensitive data inside secure views. Capture returns tokens; display resolves tokens inside the component.

## Environments and amounts

Sandbox Multi calls use `https://sandbox.weavr.io/multi`; live uses `https://secure.weavr.io/multi`. The sandbox-only simulator uses `https://sandbox.weavr.io/simulate/api`.

Monetary amounts use integer minor units: `1000` is GBP 10.00, `2550` is EUR 25.50, and `10000` is USD 100.00. Apply each currency's denomination rules rather than assuming every currency has two decimal places.
