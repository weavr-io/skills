# Weavr Skills

AI agent skills for integrating with the Weavr embedded finance platform. Works with Claude Code, Cursor, GitHub Copilot, and 37+ other agents via [skills.sh](https://skills.sh).

## Install

```bash
# Install all Weavr skills
npx skills add weavr-io/skills

# Install a specific skill
npx skills add weavr-io/skills --skill weavr-api

# Update to latest
npx skills update
```

## Available skills

| Skill                                                       | Description                                                   |
| ----------------------------------------------------------- | ------------------------------------------------------------- |
| [weavr-api](./weavr-api/SKILL.md)                           | Weavr API endpoints, authentication, request/response formats |
| [weavr-simulator](./weavr-simulator/SKILL.md)               | Sandbox testing - simulate deposits, purchases, KYC/KYB       |
| [weavr-components](./weavr-components/SKILL.md)             | Secure UI components, tokenization, integration flows         |
| [weavr-glossary](./weavr-glossary/SKILL.md)                 | Platform terminology and concepts                             |
| [weavr-sdk-web](./weavr-sdk-web/SKILL.md)                   | Web SDK setup, components, styling                            |
| [weavr-sdk-android](./weavr-sdk-android/SKILL.md)           | Android SDK, Gradle config, biometrics                        |
| [weavr-sdk-ios](./weavr-sdk-ios/SKILL.md)                   | iOS SDK, SPM/CocoaPods, biometrics                            |
| [weavr-sdk-react-native](./weavr-sdk-react-native/SKILL.md) | React Native/Expo, push provisioning                          |

## Contributing

Skills are maintained in the [docs-weavr-io](https://github.com/weavr-io/docs-weavr-io) repository and published automatically on merge to `main`.

- **Curated skills** are written by the team. Mark with `curated: true` in frontmatter.
- **Generated skills** are synthesised from documentation source by `scripts/generate-skills.js`.

To validate skills locally:

```bash
node scripts/validate-skills.js
```
