---
name: weavr-sdk-react-native
description: Integrate Weavr Secure UI and push provisioning into React Native or Expo apps. Covers Expo Modules, native dependencies, tokenized components, biometric setup, and Apple/Google Wallet provisioning. Use when building cross-platform Weavr integrations.
curated: true
version: 1.1.0
---

# Weavr React Native SDK

Documentation: https://docs.weavr.io/sdks/react-native/

## Install Secure Components

The SDK is an Expo Module and needs the Expo Modules runtime even in bare React Native apps. This doesn't require migrating the app to managed Expo.

```bash
# Bare React Native, when Expo Modules aren't wired up yet:
npx install-expo-modules
npm install expo @weavr-io/secure-components-react-native
```

Expo Go cannot load the native SDK dependencies. Use a native build or a custom development client, and rebuild after native configuration changes. Check [versions](https://docs.weavr.io/sdks/react-native/versions/) and the selected release's compatibility table instead of assuming broad React Native/Expo version ranges.

## Native configuration

Every integration needs native dependencies, regardless of the features used:

- iOS: preserve existing Podfile sources, add `https://github.com/sumsub/Specs.git` alongside the CocoaPods CDN, and add `pod 'IdensicMobileSDK', '~> 1.41.0'` in the app target.
- Pin CocoaPods to `1.17.0` or later through a Gemfile and use `bundle exec pod install`, especially when the app and a Wallet Extension share Weavr frameworks. Older embedding behavior can cause launch-time `dyld` errors.
- Android: keep `google()` and `mavenCentral()`, and add the Sumsub, Okay/GitLab, Flutter, and legacy repository entries from [Get started](https://docs.weavr.io/sdks/react-native/get-started/). Redirect all `org.bouncycastle` dependencies to `org.bouncycastle:bcprov-jdk15to18:1.68` using `resolutionStrategy.eachDependency`.
- Add OS permissions only for features used. Firebase notification/integrity setup applies to biometric features; it isn't a universal prerequisite for every secure component.
- Managed Expo: configure the SDK plugin and `expo-build-properties` per Get started. Rebuild the development client after plugin changes.
- Xcode 26 currently needs the documented `fmt`/`consteval` Podfile workaround. Use [the troubleshooting guide](https://docs.weavr.io/sdks/react-native/errors#xcode-26-build-failure) rather than changing unrelated dependencies.

## Initialize and associate

```typescript
import {
  initializeUXComponents,
  initializePSA,
  setAppCheckToken,
  setUserToken,
} from "@weavr-io/secure-components-react-native";

await initializeUXComponents(env, uiKey);
// For biometric features, with the documented environment and Firebase setup:
setAppCheckToken(appCheckToken);
await initializePSA(env);
// After obtaining an identity-scoped Access Token:
await setUserToken(accessToken);
```

Use a valid App Check token before sensitive operations and check the boolean acceptance result. Handle `AUTH` versus `ACCESS` from login; a method parameter named `authToken` doesn't imply an Auth Token is valid for financial operations.

## Secure components

| Purpose | Component or method |
| ------- | ------------------- |
| Password | `SecurePasswordTextField` |
| Token creation | `ref.current.createToken()` with `onCreateToken` |
| Passcode | `SecurePasscodeTextField`, `SecureSegmentedPasscodeTextField` (Android authentication only) |
| PAN display | `SecureCardNumberLabel` |
| CVV display | `SecureCardCVVLabel` |
| PIN display | `SecureCardPinLabel` |

Associate a stepped-up Access Token before mounting any card component. Supply the tokenized value as `toDetokenize`, hold a ref, and call `deTokenize()` to reveal it. Don't pass `cardId` and `accessToken` as replacements for the tokenized value. Follow the [card examples](https://docs.weavr.io/sdks/react-native/card-components/) for callbacks and ref handling.

Use `isVisibilityButtonEnabled` for the optional password visibility toggle. `copyOnClick` and `copyOnLongClick` on card display labels enable clipboard copying, which is off by default and writes plain text to the system clipboard.

Passcode authentication is Android-only; use password on iOS. Component styling props differ by platform. KYB is web-only, so corporate onboarding needs a web handoff.

## Push provisioning

Install the separate wallet package:

```bash
npm install @weavr-io/push-provisioning-react-native
```

Arrange Weavr configuration first. For React Native push provisioning 4.x, the iOS pod tag is `2.0.0`, not the React Native package version. Later release notes identify changes to that mapping. Install the private pod with the supplied token kept out of source control.

Include the extensionless `provisioning_config` in `android/app/src/main/res/raw/` and in the iOS app bundle plus relevant Wallet Extension targets. Add Android application metadata `com.google.android.gms.wallet.api.enabled=true`. Follow [setup](https://docs.weavr.io/sdks/react-native/push-provisioning/setup/) for entitlements, App Groups, signing, and native dependencies.

```typescript
import {
  initializeWeavrSDK,
  isGooglePayAvailable,
  getCardStatusInWallet,
} from "@weavr-io/push-provisioning-react-native";

initializeWeavrSDK({ uiKey }); // Synchronous
const googlePayAvailable = isGooglePayAvailable(); // Synchronous, for Android
const state = await getCardStatusInWallet(lastFour, "either");
const canShowButton = state === "notAdded" || state === "requiresActivation";
```

Import `AddToWalletButton` from the push provisioning package. Its integration uses `authenticationToken`, `cardId`, `cardLastFour`, `cardHolderName`, `cardDescription`, `iOSButtonStyle`, `androidAssetSource`, and `onComplete({ error, value })`. Give it explicit dimensions and display it only when the wallet is available and the card state permits it.

For a custom trigger, subscribe with `onCardProvisioningResult(cardId, callback)` before calling `addCardToWallet`, and unsubscribe on unmount. Use the exact argument order and event signatures from [implementation](https://docs.weavr.io/sdks/react-native/push-provisioning/implementation/).
