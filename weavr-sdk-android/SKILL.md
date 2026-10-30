---
name: weavr-sdk-android
description: Integrate Weavr Secure UI into native Android apps. Covers Gradle dependencies, initialization, secure capture and display, KYC, biometric authentication, and Google Wallet push provisioning. Use when building Android Weavr integrations.
curated: true
version: 1.1.0
---

# Weavr Android SDK

Documentation: https://docs.weavr.io/sdks/android/

## Install

Select a supported SDK release from the [version guide](https://docs.weavr.io/sdks/android/versions/). Use its platform requirements instead of assuming a minimum SDK or Kotlin version from an older sample.

```groovy
dependencies {
    implementation 'io.weavr.components:secure-components:<latest_version>'
}

repositories {
    google()
    mavenCentral()
    maven { url "https://maven.sumsub.com/repository/maven-public/" }
}

configurations.all {
    resolutionStrategy.eachDependency { details ->
        if (details.requested.group == "org.bouncycastle") {
            details.useTarget("org.bouncycastle:bcprov-jdk15to18:1.68")
        }
    }
}
```

If R8 full mode causes release issues, use `android.enableR8.fullMode=false` as documented. For mixed Approov versions, inspect `releaseRuntimeClasspath` with Gradle dependency insight and exclude the older transitive dependency when needed.

## Initialize

Initialize in `Application.onCreate()` so components work after process restoration:

```kotlin
UXComponents.initialize(
    context = this,
    env = ENV.SANDBOX,
    uiKey = "YOUR_UI_KEY"
)
```

Use `ENV.PRODUCTION` for live. Initialize PSA separately with `UXComponents.psa.initialize(context, psaENV, logger)` only for biometric features, following the [setup signature](https://docs.weavr.io/sdks/android/get-started/).

## Capture and display

| Purpose | Class |
| ------- | ----- |
| Password | `SecurePasswordEditText` |
| Existing passcode integrations | `SecurePasscodeEditText`, `SecurePasscodeEditTextSegmented` |
| OTP | `SecureOTPField` |
| PAN display | `SecureCardNumberTextView` |
| CVV display | `SecureCVVTextView` |
| PIN display | `SecureCardPinTextView` |
| PIN capture | `SecureCardPinEditText` |

Use the [login component guide](https://docs.weavr.io/sdks/android/login-components/) for token creation and listener signatures. Group password fields with `UXComponents.createGroup` and `match()` for confirmation. `isVisibilityButtonEnabled` or `app:showVisibilityButton` enables the password visibility toggle; it doesn't expose a plaintext accessor.

All card components require a stepped-up Access Token associated through `UXComponents.setUserToken(...)`. Fetch tokenized card fields through your server and pass them to `setSecureText(...)` using the [card component signatures](https://docs.weavr.io/sdks/android/card-components/). Don't substitute a card ID for a tokenized PAN, CVV, or PIN.

Mount or reveal only when requested by the cardholder. Clipboard copying is off by default; enabling it writes plain text to the system clipboard.

## Verification and biometrics

KYC uses Sumsub. Configure gallery upload types with the `sns_gallery_type` string resource; `*/*` allows all file types. Corporate KYB is web-only, so hand corporate onboarding to a web flow.

Biometrics require programme configuration, SDK/PSA initialization, and the documented device-integrity and notification setup. Use `UXComponents.psa.startEnrollment` and `UXComponents.psa.startBiometricPsaLogin`, with the result/listener types from the [enrollment and login guide](https://docs.weavr.io/sdks/android/biometrics-enrollment-login/). Prefer password fallback for new integrations. Handle Auth Token to Access Token exchange rather than assuming a biometric callback is an identity-scoped session.

## Google Wallet

Push provisioning has its own SDK setup and requires Weavr configuration before code can work. Follow [push provisioning setup](https://docs.weavr.io/sdks/android/push-provisioning/setup/) and [implementation](https://docs.weavr.io/sdks/android/push-provisioning/implementation/); check wallet/card availability before offering provisioning.
