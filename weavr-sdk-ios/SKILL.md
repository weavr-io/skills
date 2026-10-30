---
name: weavr-sdk-ios
description: Integrate Weavr Secure UI into native iOS apps. Covers Swift Package Manager and CocoaPods setup, asynchronous initialization, secure capture and display, KYC, biometrics, and Apple Wallet provisioning. Use when building iOS Weavr integrations.
curated: true
version: 1.1.0
---

# Weavr iOS SDK

Documentation: https://docs.weavr.io/sdks/ios/

## Install

Swift Package Manager is recommended. Add:

```text
https://github.com/weavr-io/secure-components-ios.git
```

Choose a supported release from the [version guide](https://docs.weavr.io/sdks/ios/versions/), with an iOS deployment target of at least 15.1.

For CocoaPods, preserve existing sources and add Sumsub's spec repository:

```ruby
source 'https://cdn.cocoapods.org/'
source 'https://github.com/sumsub/Specs.git'
platform :ios, '15.1'
use_frameworks! :linkage => :dynamic

pod 'ApproovURLSession', '~> 3.3.3'
pod 'WeavrComponents', '~> 3.0'
pod 'IdensicMobileSDK', '~> 1.41.0'
pod 'Firebase/Analytics'
pod 'Firebase/Core'
pod 'Firebase/Messaging'
pod 'FirebaseAppCheck'
```

Match dependency versions to the SDK release being integrated. Configure the documented camera, photo library, microphone, location, and Face ID usage descriptions for verification and biometric features.

## Initialize

Start initialization during the app's launch lifecycle and await the result before mounting components:

```swift
import WeavrComponents

let result = await UXComponents.initialize(
    environment: .Sandbox,
    uiKey: "YOUR_UI_KEY"
)
```

Handle every `InitializationResult` case: `.initialized`, `.serverError`, `.networkFailure`, `.setupError`, and `.failure`. On success, inspect `userNeedsRelogin` and require a fresh login when set. Preserve `traceability` for support; `authenticationType` describes the project's configured authentication mode. Use the exact [result signatures](https://docs.weavr.io/sdks/ios/get-started/) for the chosen release.

Retrieve a valid Firebase App Check token before sensitive login or enrollment operations and pass it to `UXComponents.setAppCheckToken(token)`. Check its `Bool` result; an empty string isn't a production token.

## Capture and display

| Purpose | Class or API |
| ------- | ------------ |
| Password | `SecurePasswordField` |
| Password confirmation | Collection of fields with `secureInputsMatch` |
| Credential tokenization | `await field.tokenise()` |
| PAN display | `SecureCardNumberLabel` |
| CVV display | `SecureCVVLabel` |
| PIN display | `SecureCardPinLabel` |
| PIN capture | `SecureCardPinField` |

Use passwords for iOS authentication; numeric authentication passcodes aren't permitted. An optional `isVisibilityButtonEnabled` toggle reveals the password visually without exposing a plaintext accessor.

After login, associate the Access Token using `await UXComponents.setUserToken(token: token)` and handle `AssociateTokenResult`. All card components require step-up. Fetch tokenized values from your server and use `detokenise(_:)` with the [documented result handling](https://docs.weavr.io/sdks/ios/card-components/). These labels don't take `cardId` and `accessToken` properties as a substitute for tokenized values.

Reveal sensitive fields only on request and remove or hide them promptly afterward. Clipboard copying is opt-in and places plain text outside the secure view.

## KYC and biometrics

Consumer KYC uses the native verification component; corporate KYB is web-only and needs a web handoff.

Complete the [biometric setup](https://docs.weavr.io/sdks/ios/biometrics/) before enrollment, including programme, notification, and integrity configuration. After the main SDK initializes, call `UXComponents.psa.initialize()` and keep the FCM token current through `UXComponents.psa.updateFCMToken(...)`. Use `UXComponents.psa.startEnrollment(viewController:)` for enrollment and `await UXComponents.psa.startBiometricLogin(viewController:)` for login, following the [enrollment and login guide](https://docs.weavr.io/sdks/ios/biometrics-enrollment-login/). Biometric login requires exchanging the Auth Token for an identity-scoped Access Token. Provide password fallback.

## Apple Wallet

Push provisioning has separate SDK dependencies, entitlement approval, configuration, and Wallet Extension requirements. Arrange Weavr setup first, then follow [push provisioning setup](https://docs.weavr.io/sdks/ios/push-provisioning/setup/) and [implementation](https://docs.weavr.io/sdks/ios/push-provisioning/implementation/). Don't assume installing Secure Components enables wallet provisioning.
