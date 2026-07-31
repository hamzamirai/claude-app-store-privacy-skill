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
| `Docs/AI-PRIVACY-DISCLOSURE.md` | Branded AI disclosure sheet (generated on request, styled with app theme & colors) |
| `Shared/Views/Settings/AIPrivacyDisclosureView.swift` | Ready-to-use SwiftUI view — branded disclosure sheet with hero header, compliance badge, and disclosure cards matching the app's color system |
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

```bash
npm install -g docx
```

**The skill always works without it** — you'll still get `PrivacyInfo.xcprivacy` files, the full privacy analysis, age rating answers, compliance checks, AI provider disclosure text, and the `AIPrivacyDisclosureView.swift` SwiftUI view. The optional `docx` skill upgrades the report to a Word document.

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
HealthKit, CoreLocation, Contacts, ARKit, RealityKit, AVFoundation, PhotosUI, Speech, LocalAuthentication, UserNotifications, PassKit, AdSupport, AppTrackingTransparency, CloudKit, CoreData, SwiftData, SensitiveContentAnalysis, ActivityKit, CoreMediaIO, EnergyKit, DeclaredAgeRange

### AI & Machine Learning
Apple Intelligence (Foundation Models, Writing Tools, Image Playground), OpenAI (GPT-4 / GPT-4o), Google Gemini (free tier), Firebase AI / Gemini (paid tier, formerly Vertex AI for Firebase), Anthropic Claude, Mistral, Groq, Cohere, Perplexity, CoreML, NaturalLanguage

### Platform-Specific
visionOS: HandTrackingProvider, WorldTrackingProvider, SceneReconstructionProvider, MeshAnchor
watchOS: HKWorkoutSession, CMMotionManager, WKExtendedRuntimeSession
macOS: CGWindowListCopyWindowInfo, NSWorkspace, SCNetworkReachability
tvOS: TVUserManager, TVTopShelfContentProvider

## App Store Compliance Checks

Beyond privacy declarations, the skill also runs 42 pre-submission checks:

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
| UGC safety | §1.2 | Chat/UGC detected → moderation + developer responsibility |
| Kids/teen category restrictions | §1.3 | Banned APIs, analytics, and age verification |
| Subscription management link | §3.1.2 | Auto-renewable subscription → manage/cancel flow |
| Privacy policy URL | §5.1.1 | NSPrivacyPolicyURL in Info.plist |
| Placeholder content | §2.1 | No TODO/Lorem ipsum before submission |
| Family Controls entitlement | §3.3.3 | FamilyControls → Apple-approved entitlement required |
| Foveated streaming disclosure | §3.3.3(B) | Eye tracking data must be declared |
| Accessory notifications scope | §3.3.7(J) | System-managed accessory setup verification |
| Brazil storefront A12 | Mar 2026 | Advertising/chat apps auto-rated A12 in Brazil |
| No ads in extensions/widgets/watchOS | §2.5.18 | Ads only in main app binary |
| macOS: App Sandbox | §2.4.5(i) | Sandbox entitlement required for Mac App Store |
| macOS: No third-party updater | §2.4.5(vii) | Sparkle/DevMateKit prohibited |
| HealthKit + ad/analytics isolation | §5.1.3(i) | Health data must not power ad targeting |
| Location + advertising separation | §5.1.5 | Location data not allowed for ad targeting |
| VPN entitlement | §5.4 | NetworkExtension → Apple-approved entitlement |
| MDM entitlement | §5.5 | MDM restricted to enterprise/educational use |
| Medical app disclaimer | §1.4.1 | HealthKit/medical terms → consult-doctor disclaimer |
| Export compliance and developer identity | §3.1 / §14.8 | ITSAppUsesNonExemptEncryption must match detected cryptography |
| Private / deprecated API usage | §2.5.1 | No private frameworks, UIWebView, UIAlertView |
| Loot box odds disclosure | §3.1.1 | Randomized IAP → odds must be disclosed |
| Facial recognition → LocalAuthentication | §2.5.13 | ARKit face tracking + auth → use LocalAuthentication |
| Cryptocurrency mining | §2.4.2 / §3.1.5(ii) | On-device crypto mining prohibited |
| CallKit proper use | §2.5.12 | Call data not shared with third parties |
| SensitiveContentAnalysis entitlement | §3.3.3(N) | Sensitive Content Analysis → Apple-approved entitlement |
| Suggested Actions privacy disclosure | §3.3.3(Q) | Suggested Actions exposing user data → privacy manifest |
| Trust Insights data handling | §3.3.3(R) | Trust signals not shared with third parties |
| Media Device Extension entitlement | §3.3.7(L) | CoreMediaIO extensions → camera entitlement required |
| Spatial Audio Extension registration | §3.3.7(M) | Spatial AU extensions must declare AudioComponents |
| Customer Engagement APIs | §3.3.9(E) | Engagement APIs for existing customers only |
| Live Activities: no spam/phishing | §4.5.3 | Live Activities must not contain promotional/spam content |
| Foundation Models license compliance | §3.3.11(A) | Foundation Models outputs not falsely attributed to humans |
| Apple models access terms | §3.2(h) | Apple model output not used to train or improve another model |
| Accessibility content modification | §3.3.4(A) | Dynamic Type, VoiceOver, and appearance settings not blocked |
| Passes privacy handling | Att. 5 §3.3 | Wallet pass data not shared with third parties |
| EnergyKit identity guidelines | Att. 11 §4 | EnergyKit entitlement + correct Apple attribution |
| Spam / duplicate apps | §4.3(a) / §4.3(b) | No clone targets, reskins, or template placeholders |
| Kid & teen safety age assurance | Intro / §7.9 | Age assurance for minor-reachable social and paid features |

## AI Privacy Features

When AI SDKs are detected, the skill:

- Identifies each provider's data policy (retention period, training opt-out, server location)
- Determines what App Store Connect data types to declare per §5.1.2(i)
- Generates ready-to-paste privacy policy language per provider
- Generates in-app consent disclosure text
- Generates a ready-to-use `AIPrivacyDisclosureView.swift` — a branded SwiftUI view styled with the app's color system (hero header, compliance badge, disclosure cards)

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
## Compliance: 36/37 checks passed — 1 WARN (account deletion not found)
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
