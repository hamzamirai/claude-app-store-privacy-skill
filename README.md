![App Privacy Skill Hero](assets/hero.png)

# App Privacy & Age Rating Skill for Claude Code

Automatically scan your Apple platform Xcode project and generate complete App Store Privacy Details, Age Rating answers, AI Privacy disclosures, compliance findings, and a professional `.docx` report — all from your codebase.

## What It Does

1. **Scans** your project for privacy-relevant SDKs, frameworks, imports, code patterns, and AI APIs
2. **Detects target platforms** (iOS, macOS, visionOS, watchOS, tvOS) and applies platform-specific checks
3. **Maps** findings to Apple's privacy taxonomy (14 data categories, 30+ data types, 6 purposes)
4. **Answers** the Age Rating questionnaire based on detected content (violence, medical, advertising, UGC, etc.)
5. **Detects AI providers** (Apple Intelligence, OpenAI, Gemini, Claude/Anthropic, Mistral, etc.) and generates §5.1.2(i)-compliant disclosures
6. **Runs compliance checks** against Apple Review Guidelines (ATT parity, account deletion, Restore Purchases, Apple Sign In, external payments)
7. **Asks** clarifying questions for ambiguous cases
8. **Generates** outputs in the project `Docs/` folder

## Outputs

| File | Description |
|------|-------------|
| `Docs/APP-PRIVACY-AND-AGE-RATING-REPORT.docx` | Full report: privacy declarations, age rating, AI data practices, compliance findings |
| `Docs/AI-PRIVACY-DISCLOSURE.md` + `.pdf` | Branded AI disclosure sheet (generated on request, styled with app theme & colors) |
| `<target>/PrivacyInfo.xcprivacy` | Apple privacy manifest — one per platform target |

## Supported Platforms

- iOS / iPadOS
- macOS
- visionOS (with spatial, hand tracking, and scene reconstruction detection)
- watchOS (with workout, motion, and complication data detection)
- tvOS (with multi-user profile detection)

## Prerequisites

This skill has **two optional dependencies** that unlock richer output. The core privacy scan, PrivacyInfo.xcprivacy generation, and compliance checks work without them.

| Skill | What it unlocks | Without it |
|-------|----------------|------------|
| **`docx`** | `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` Word document | Falls back to markdown report only |
| **`canvas-design`** | Branded `AI-PRIVACY-DISCLOSURE.pdf` with app colors & icon | Falls back to plain `.md` file only |

Both are optional — install only what you need:

```bash
# Install docx skill (for Word document report)
# Get from: https://github.com/anthropics/anthropic-skills
cp -r /path/to/anthropic-skills/skills/docx ~/.claude/skills/

# Install canvas-design skill (for branded AI disclosure PDF)
# Get from: https://github.com/anthropics/anthropic-skills
cp -r /path/to/anthropic-skills/skills/canvas-design ~/.claude/skills/
```

**The skill always works without either** — you'll still get `PrivacyInfo.xcprivacy` files, the full privacy analysis, age rating answers, compliance checks, and AI provider disclosure text. The optional skills just upgrade the output format.

## Installation

```bash
# Global installation (available in all projects)
cp -r app-privacy ~/.claude/skills/

# Project-level installation (available only in this project)
cp -r app-privacy /path/to/your/project/.claude/skills/
```

## Usage

In Claude Code, say any of:

- `"Generate app privacy details for this project"`
- `"Scan this app for privacy data collection"`
- `"What privacy data does this app collect?"`
- `"Help me fill out the App Store privacy section"`
- `"Generate a PrivacyInfo.xcprivacy"`
- `"Run a privacy audit"`
- `"What's the age rating for this app?"`
- `"App Store submission report"`

## Supported SDKs & Frameworks

### Privacy & Analytics
Firebase (Analytics, Auth, Crashlytics, Firestore, Storage, Messaging, Remote Config), RevenueCat, StoreKit, Google Sign-In, Google AdMob, Facebook SDK, Mixpanel, Amplitude, Segment, Sentry, TelemetryDeck, PostHog, OneSignal, Braze, Leanplum

### Attribution
Adjust, AppsFlyer, Branch, Singular, Kochava

### Platform Frameworks
HealthKit, CoreLocation, Contacts, ARKit, RealityKit, AVFoundation, PhotosUI, Speech, LocalAuthentication, UserNotifications, PassKit, AdSupport, AppTrackingTransparency, CloudKit, CoreData, SwiftData

### AI & Machine Learning
Apple Intelligence (Foundation Models, Writing Tools, Image Playground), OpenAI (GPT-4 / GPT-4o), Google Gemini (free tier), Firebase AI / Gemini (paid tier, formerly Vertex AI for Firebase), Anthropic Claude, Mistral, Groq, Cohere, Perplexity, CoreML, NaturalLanguage

### Platform-Specific
visionOS: HandTrackingProvider, WorldTrackingProvider, SceneReconstructionProvider, MeshAnchor
watchOS: HKWorkoutSession, CMMotionManager, WKExtendedRuntimeSession
macOS: CGWindowListCopyWindowInfo, NSWorkspace, SCNetworkReachability
tvOS: TVUserManager, TVTopShelfContentProvider

## App Store Compliance Checks

Beyond privacy declarations, the skill also runs these pre-submission checks:

| Check | Guideline | What It Verifies |
|-------|-----------|-----------------|
| ATT for tracking SDKs | §5.1.2 | Ad SDKs present → ATT must be implemented |
| Purpose string quality | §5.1.1 | NSUsageDescription strings must be specific, not vague |
| Account deletion | §5.1.1 | If auth detected → delete account option must exist |
| Restore Purchases | §3.1.1 | If IAP detected → restore flow must be implemented |
| External payment | §3.1.1 | Stripe/PayPal for digital goods = critical rejection |
| Apple Sign In parity | §4.8 | Social login → Apple Sign In must also be offered |
| Third-party AI disclosure | §5.1.2(i) | Cloud AI usage → must disclose to users |
| Gemini free tier training | §5.1.2(i) | Free tier trains on data — disclose or upgrade |
| Chatbot age restriction | §4.7.5 | AI chat features need age-restriction mechanisms |

## AI Privacy Features

When AI SDKs are detected, the skill:

- Identifies each provider's data policy (retention period, training opt-out, server location)
- Determines what App Store Connect data types to declare per §5.1.2(i)
- Generates ready-to-paste privacy policy language per provider
- Generates in-app consent disclosure text
- Asks if you want a branded `AI-PRIVACY-DISCLOSURE.pdf` styled with your app's colors and icon

### Provider Policies at a Glance

| Provider | Trains on API Data | Retention | Server |
|----------|-------------------|-----------|--------|
| Apple Intelligence (on-device) | No | N/A — on-device | On-device |
| OpenAI API | No (API default) | 30 days | US |
| Google Gemini (free tier) | **Yes** | Unspecified | Google servers |
| Google Gemini (paid tier) | No | Limited | Google servers |
| Anthropic Claude | Opt-out available | 30 days | US |

## Example Output Summary

```
## Privacy Summary
| Category                   | Count   |
|----------------------------|---------|
| Data Used to Track You     | 0 types |
| Data Linked to You         | 4 types |
| Data Not Linked to You     | 3 types |
| Total Data Types Collected | 7 types |

## Age Rating: 4+

## AI Providers Detected: Apple Intelligence (on-device), OpenAI
## Compliance: 8/9 checks passed — 1 WARN (account deletion not found)
```

## Limitations

- Cannot detect server-side-only data collection (e.g., IP logging, server analytics)
- Cannot determine backend data linking without understanding your server architecture
- May miss custom/proprietary analytics implementations
- SDK privacy requirements may change with SDK updates — verify against latest docs
- Does not analyze WKWebView third-party content
- Age rating content detection is best-effort — always verify manually for content-heavy apps
- AI provider policies change frequently — re-run before every submission

## Re-run When

- You add, remove, or update SDKs/frameworks
- You integrate a new AI provider or change AI feature scope
- Before every App Store submission

## License

MIT
