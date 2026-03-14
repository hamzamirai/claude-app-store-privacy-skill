---
name: app-privacy
description: Scan SwiftUI/iOS codebases to detect privacy-relevant SDKs, frameworks, AI APIs (Apple Intelligence, OpenAI, Gemini, Claude/Anthropic, Mistral), and data collection patterns. Generates App Store Privacy Details, Age Rating answers, PrivacyInfo.xcprivacy per platform target, AI-PRIVACY-DISCLOSURE.md for §5.1.2(i) compliance, App Store compliance findings, and a professional .docx report (or Markdown, or both — user's choice) saved to Docs/
---

# App Privacy & Age Rating Skill

You are an expert in Apple's App Store Privacy requirements and Age Rating system. When invoked, you scan the user's Xcode project to detect all privacy-relevant SDKs, frameworks, and data collection patterns, then generate a complete App Privacy declaration, Age Rating questionnaire answers, and a professional .docx report.

**Supported Platforms**: iOS, iPadOS, macOS, visionOS, watchOS, tvOS — the skill detects which platforms the app targets and adjusts detection accordingly.

## Trigger Phrases

Activate this skill when the user says any of:
- "Generate app privacy details"
- "Scan for privacy data"
- "What privacy data does this app collect?"
- "Help me fill out the App Store privacy section"
- "Generate a PrivacyInfo.xcprivacy"
- "App privacy report"
- "Privacy audit"
- "Age rating"
- "App Store submission report"
- Or any reference to App Store privacy declarations, privacy nutrition labels, privacy manifests, or age rating questionnaires

## Execution Flow

Follow these 11 phases in order. Do NOT skip phases.

---

### Phase 1: Project Discovery

Scan the project root to identify the project structure and dependency management:

1. **Use Glob** to find:
   - `**/Package.swift` (Swift Package Manager)
   - `**/Podfile` (CocoaPods)
   - `**/Cartfile` (Carthage)
   - `**/*.xcodeproj/project.pbxproj` (Xcode project)
   - `**/*.xcworkspace` (Xcode workspace)
   - `**/PrivacyInfo.xcprivacy` (existing privacy manifests)
   - `**/Info.plist` (platform and entitlements info)

2. **Read** each found file to extract:
   - SPM: `.package(url:` entries → dependency names and URLs
   - CocoaPods: `pod '...'` entries → pod names
   - Carthage: dependency names
   - Existing privacy manifests: current declarations

3. **Detect target platforms** by scanning:
   - `project.pbxproj` for `SDKROOT` and `SUPPORTED_PLATFORMS` (iphoneos, macosx, appletvos, watchos, xros)
   - `Package.swift` for `.iOS`, `.macOS`, `.visionOS`, `.watchOS`, `.tvOS` platform declarations
   - `#if os(iOS)`, `#if os(macOS)`, `#if os(visionOS)`, `#if os(watchOS)`, `#if os(tvOS)` conditional compilation blocks
   - `.destination` or target names containing "Watch", "TV", "Vision", "Mac"

4. **Report** what was found: project name, target platform(s), dependency manager(s), number of dependencies detected.

---

### Phase 2: Dependency & Import Detection

1. **Grep all `.swift` files** for import statements matching privacy-relevant frameworks. Search for each of these patterns:

```
import Firebase
import FirebaseAnalytics
import FirebaseAuth
import FirebaseFirestore
import FirebaseStorage
import FirebaseCrashlytics
import FirebaseMessaging
import FirebaseRemoteConfig
import GoogleSignIn
import GoogleMobileAds
import RevenueCat
import StoreKit
import HealthKit
import CoreLocation
import MapKit
import Contacts
import ContactsUI
import AdSupport
import AppTrackingTransparency
import UserNotifications
import AVFoundation
import PhotosUI
import Photos
import Speech
import ARKit
import RealityKit
import CoreML
import NaturalLanguage
import LocalAuthentication
import AuthenticationServices
import FacebookCore
import FBSDKCoreKit
import Mixpanel
import Amplitude
import Segment
import Sentry
import OneSignal
import Braze
import Leanplum
import Adjust
import AppsFlyer
import Branch
import Singular
import Kochava
import TelemetryDeck
import PostHog
```

#### AI & Machine Learning SDKs

Also grep for AI/ML SDK imports:

```
import FoundationModels
import OpenAI
import OpenAIKit
import OpenAISwift
import GoogleGenerativeAI
import FirebaseAI
import Anthropic
import AIProxy
import CoreML
import CreateML
import NaturalLanguage
import SoundAnalysis
```

Also grep for AI provider endpoint strings in any `.swift` file:
```
api.openai.com
generativelanguage.googleapis.com
api.anthropic.com
api.mistral.ai
api.groq.com
api.cohere.com
api.perplexity.ai
aiplatform.googleapis.com
firebaseai.googleapis.com
```

2. **Record** every unique import found with the file path where it was detected.

---

### Phase 3: Code Pattern Detection

**Grep all `.swift` files** for the following privacy-relevant code patterns. For each match, record the file path and line number.

#### Location
| Pattern | Implication |
|---------|-------------|
| `CLLocationManager` | Location data access |
| `CLLocationCoordinate2D` | Location coordinates used |
| `requestWhenInUseAuthorization` | Location permission requested |
| `requestAlwaysAuthorization` | Always-on location |
| `startUpdatingLocation` | Active location tracking |
| `MKMapView` | Map display (may use location) |

#### Health & Fitness
| Pattern | Implication |
|---------|-------------|
| `HKHealthStore` | HealthKit data access |
| `HKQuantityType` | Health quantity data |
| `HKCategoryType` | Health category data |
| `HKWorkout` | Workout/fitness data |
| `HKElectrocardiogram` | Clinical health data |

#### Contacts
| Pattern | Implication |
|---------|-------------|
| `CNContactStore` | Contacts access |
| `CNContact` | Contact data usage |
| `CNContactPickerViewController` | Contact picker UI |

#### Identifiers & Tracking
| Pattern | Implication |
|---------|-------------|
| `ASIdentifierManager` | Advertising identifier |
| `advertisingIdentifier` | IDFA access |
| `identifierForVendor` | Device ID (IDFV) |
| `ATTrackingManager` | App Tracking Transparency |
| `requestTrackingAuthorization` | Tracking permission |

#### Authentication & User Identity
| Pattern | Implication |
|---------|-------------|
| `ASAuthorizationAppleIDProvider` | Apple Sign In |
| `ASAuthorizationAppleIDCredential` | Apple ID credential |
| `GIDSignIn` | Google Sign In |
| `Auth.auth()` | Firebase Auth |
| `createUser(withEmail` | Email/password registration |
| `signIn(withEmail` | Email/password login |
| `OAuthProvider` | OAuth authentication |

#### Analytics & Diagnostics
| Pattern | Implication |
|---------|-------------|
| `Analytics.logEvent` | Firebase Analytics event |
| `Analytics.setUserProperty` | Firebase user property |
| `Analytics.setUserID` | Firebase user ID tracking |
| `Crashlytics.crashlytics()` | Crash reporting |
| `Crashlytics.setUserID` | Crash report user linking |

#### Purchases & Payments
| Pattern | Implication |
|---------|-------------|
| `Purchases.shared` | RevenueCat purchases |
| `Purchases.configure` | RevenueCat initialization |
| `Product.products` | StoreKit 2 products |
| `Transaction.currentEntitlements` | StoreKit 2 transactions |
| `SKPaymentQueue` | StoreKit 1 purchases |
| `PKPaymentAuthorizationController` | Apple Pay |

#### User Content
| Pattern | Implication |
|---------|-------------|
| `AVCaptureSession` | Camera/microphone capture |
| `AVCaptureDevice` | Camera/mic device access |
| `PHPhotoLibrary` | Photo library access |
| `PHPickerViewController` | Photo picker |
| `PhotosPickerItem` | SwiftUI photo picker |
| `SFSpeechRecognizer` | Speech recognition |
| `UIImagePickerController` | Image picker (legacy) |

#### Surroundings & Body (visionOS / ARKit)
| Pattern | Implication | Platform |
|---------|-------------|----------|
| `ARSession` | AR session (surroundings) | iOS, visionOS |
| `ARWorldTrackingConfiguration` | World tracking | iOS |
| `HandTrackingProvider` | Hand tracking data | visionOS |
| `WorldTrackingProvider` | World tracking | visionOS |
| `MeshAnchor` | Environment mesh | visionOS |
| `PlaneAnchor` | Plane detection | visionOS |
| `SceneReconstructionProvider` | Scene reconstruction | visionOS |
| `ARBodyTrackingConfiguration` | Body tracking | iOS |
| `ImmersiveSpace` | Immersive content | visionOS |

#### watchOS-Specific
| Pattern | Implication |
|---------|-------------|
| `HKWorkoutSession` | Active workout tracking |
| `CLKComplicationDataSource` | Complication data (may use health/location) |
| `WKExtendedRuntimeSession` | Background session (may collect data) |
| `HealthStore` in WatchKit context | Watch health data access |
| `CMMotionManager` | Motion/accelerometer data |

#### macOS-Specific
| Pattern | Implication |
|---------|-------------|
| `NSAppleScript` | Script execution (potential data access) |
| `NSWorkspace` | System info access |
| `SCNetworkReachability` | Network status |
| `IOKit` | Hardware info access |
| `NSScreen` | Display info |
| `CGWindowListCopyWindowInfo` | Window list access (privacy-sensitive) |
| `TCC` / `tccd` references | Privacy permission checks |

#### tvOS-Specific
| Pattern | Implication |
|---------|-------------|
| `TVUserManager` | Multi-user profile management |
| `TVTopShelfContentProvider` | Top Shelf content (may use user data) |

#### AI & Machine Learning (All Platforms)
| Pattern | Implication | Provider |
|---------|-------------|----------|
| `LanguageModelSession()` | On-device foundation model | Apple Intelligence |
| `LanguageModel.default` | On-device LLM inference | Apple Intelligence |
| `WritingToolsBehavior` | Writing Tools integration | Apple Intelligence |
| `ImageCreator` | On-device image generation | Apple Intelligence |
| `ImagePlaygroundViewController` | Image Playground UI | Apple Intelligence |
| `OpenAI(apiKey:` | OpenAI client initialization — prompts sent to OpenAI | OpenAI |
| `ChatQuery(` | OpenAI chat request — user messages sent to OpenAI | OpenAI |
| `GenerativeModel(` | Gemini model instantiation — prompts sent to Google | Google Gemini |
| `model.generateContent(` | Gemini content generation request | Google Gemini |
| `client.messages.create` | Anthropic Claude API call — prompts sent to Anthropic | Claude (Anthropic) |
| `Anthropic(apiKey:` | Anthropic client initialization | Claude (Anthropic) |
| `"api.openai.com"` | Direct OpenAI API endpoint usage | OpenAI |
| `"generativelanguage.googleapis.com"` | Direct Gemini API endpoint | Google Gemini |
| `"api.anthropic.com"` | Direct Claude API endpoint | Claude (Anthropic) |
| `MLModel(` | CoreML model inference | On-device ML |
| `NLModel(` | Natural Language ML model | On-device ML |
| `"api.mistral.ai"` | Mistral AI API endpoint | Mistral |
| `"api.groq.com"` | Groq API endpoint | Groq |

#### Data Storage (Context-dependent)
| Pattern | Implication |
|---------|-------------|
| `Firestore.firestore()` | Cloud data storage — ASK user what data |
| `FirebaseStorage` | Cloud file storage — ASK user what content |
| `UserDefaults` | Local preferences (Required Reason API) |
| `@AppStorage` | SwiftUI UserDefaults wrapper |
| `NSUbiquitousKeyValueStore` | iCloud key-value storage |
| `CloudKit` | iCloud database |
| `CoreData` | Local database — may sync to cloud |
| `SwiftData` | Local database — may sync to cloud |
| `KeychainAccess` / `Security.framework` | Secure credential storage |

#### Advertising
| Pattern | Implication |
|---------|-------------|
| `GADMobileAds` | Google AdMob |
| `GADBannerView` | AdMob banner ads |
| `GADInterstitialAd` | AdMob interstitial ads |
| `GADRewardedAd` | AdMob rewarded ads |
| `SKAdNetwork` | Ad attribution |
| `AdServices` | Apple Search Ads attribution |

#### Push Notifications
| Pattern | Implication |
|---------|-------------|
| `UNUserNotificationCenter` | Local/push notifications |
| `Messaging.messaging()` | Firebase Cloud Messaging |
| `registerForRemoteNotifications` | Push notification registration |

---

### Phase 4: SDK-to-Privacy Mapping

Apply the following knowledge base to map each detected SDK/framework to Apple's privacy data types. This is the core reference table.

#### Firebase Analytics
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | No | No | Analytics |
| Product Interaction | No | No | Analytics |
| Diagnostics | No | No | Analytics |

> Note: If `Analytics.setUserID` is detected, upgrade Device ID and Product Interaction to **Linked = Yes**.

#### Firebase Crashlytics
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Diagnostics | No | No | App Functionality |

> Note: If `Crashlytics.setUserID` is detected, add **User ID — Linked = Yes**.

#### Firebase Auth (Email/Password)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Email Address | Yes | No | App Functionality |
| User ID | Yes | No | App Functionality |

#### Firebase Auth (Apple Sign In)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Name | Yes | No | App Functionality |
| Email Address | Yes | No | App Functionality |
| User ID | Yes | No | App Functionality |

#### Google Sign-In
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Name | Yes | No | App Functionality |
| Email Address | Yes | No | App Functionality |
| User ID | Yes | No | App Functionality |

#### RevenueCat
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Purchase History | Yes | No | App Functionality |
| User ID | Yes | No | App Functionality |

#### StoreKit (1 or 2)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Purchase History | Yes | No | App Functionality |

#### Google AdMob / Google Mobile Ads
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | No | Yes | Third-Party Advertising |
| Advertising Data | No | Yes | Third-Party Advertising |
| Coarse Location | No | No | Third-Party Advertising |
| Diagnostics | No | No | Third-Party Advertising |

#### Facebook SDK
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | No | Yes | Third-Party Advertising |
| Advertising Data | No | Yes | Third-Party Advertising |
| Product Interaction | Yes | Yes | Third-Party Advertising, Analytics |

#### HealthKit
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Health Data | Yes | No | App Functionality |
| Fitness Data | Yes | No | App Functionality |

> Note: HealthKit data MUST always be declared, even if processed on-device only. Apple requires this.

#### CoreLocation (Precise)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Precise Location | ASK | No | App Functionality |

> Ask user: "Is precise location data sent to a server or used only on-device?"
> If on-device only: exempt from disclosure (except if used for tracking).
> If sent to server: Linked = depends on whether tied to user identity.

#### CoreLocation (Coarse / MapKit only)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Coarse Location | No | No | App Functionality |

#### Contacts Framework
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Contacts | Yes | No | App Functionality |

> Ask user: "Are contacts uploaded to a server or used only on-device?"

#### visionOS Spatial APIs (ARKit, RealityKit)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Environment Scanning | No | No | App Functionality |
| Hands Data | No | No | App Functionality |
| Head Movement | No | No | App Functionality |

> Note: If spatial data is processed on-device only (typical for visionOS), these may be exempt. Ask user to confirm.

#### Camera / Photos / Microphone
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Photos or Videos | ASK | No | App Functionality |
| Audio Data | ASK | No | App Functionality |

> Ask user: "Are photos/videos/audio uploaded to a server or processed on-device only?"

#### Mixpanel / Amplitude / Segment / PostHog / TelemetryDeck
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | Yes | No | Analytics |
| Product Interaction | Yes | No | Analytics |

#### Sentry
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Diagnostics | No | No | App Functionality |

#### OneSignal / Braze / Leanplum
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | Yes | No | App Functionality |
| Email Address | Yes | No | App Functionality |

> Only declare Email Address if the SDK is configured with user email.

#### Attribution SDKs (Adjust, AppsFlyer, Branch, Singular, Kochava)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | No | Yes | Developer's Advertising |
| Advertising Data | No | Yes | Developer's Advertising |
| Product Interaction | No | No | Analytics |

#### Apple Pay (PassKit)
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Payment Info | No | No | App Functionality |

> Note: Apple handles payment processing. The app does not receive full card details.

#### Firebase Cloud Messaging
| Data Type | Linked | Tracking | Purpose |
|-----------|--------|----------|---------|
| Device ID | Yes | No | App Functionality |

#### OpenAI API (ChatGPT / GPT-4 / GPT-4o)
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts/messages) | ASK | No | App Functionality | Sent to OpenAI servers; retained up to 30 days |
| Other Data Types (AI-generated responses) | No | No | App Functionality | — |

> **OpenAI data policy**: API inputs/outputs are NOT used for model training by default (API users are opted out). Data is retained for up to 30 days for abuse/safety monitoring. Zero-retention (0-day) available via enterprise agreement. Always ask user: "Do prompts contain any user-identifiable information (name, email, health data, location)?"

#### Google Gemini API (Generative Language / Firebase AI)
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts/messages) | ASK | No | App Functionality | Sent to Google servers |
| Other Data Types (AI-generated responses) | No | No | App Functionality | — |

> **Gemini data policy**:
> - **Free tier (AI Studio / free quota)**: Google MAY use prompts and responses to improve models. Human reviewers may read content (de-linked from account). MUST disclose this to users.
> - **Paid tier**: NOT used for training. Retained for limited period for abuse detection only.
> - Grounding with Google Search: data retained 30 days; Maps grounding: up to 90 days.
> Always ask user: "Are you on the free or paid Gemini API tier?"

#### Anthropic Claude API
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts/messages) | ASK | No | App Functionality | Sent to Anthropic servers; retained up to 30 days |
| Other Data Types (AI-generated responses) | No | No | App Functionality | — |

> **Anthropic data policy**: API inputs/outputs are deleted within 30 days by default. May be used for model training UNLESS the developer has opted out (via API settings or enterprise agreement). Safety-flagged content may be retained up to 2 years. Always ask user: "Have you opted out of Anthropic training on your API account?" and "Do prompts contain user-identifiable data?"

#### Apple Intelligence — Foundation Models (On-Device)
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts) | No | No | App Functionality | Processed fully on-device; no data sent to Apple |

> **Apple Intelligence data policy**: `FoundationModels` framework processes all requests on-device. No data leaves the device. No disclosure required for on-device processing (same as on-device CoreML). EXCEPTION: If Private Cloud Compute (PCC) is used for overflow, Apple's privacy guarantees apply but disclosure in privacy policy is recommended.

#### Apple Intelligence — ChatGPT Extension (Opt-in)
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts routed to ChatGPT) | No | No | App Functionality | Sent to OpenAI only after explicit user opt-in per request |

> **ChatGPT extension policy**: Apple routes requests to OpenAI only with explicit per-request user consent (shown in system UI). OpenAI's privacy policy applies. IP addresses are obscured by Apple. Declare as third-party AI data sharing per §5.1.2(i).

#### Apple Intelligence — Writing Tools / Image Playground / Visual Intelligence
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content | No | No | App Functionality | Entirely on-device processing |

> No privacy declarations needed — Apple handles these features. Apps that adopt `WritingToolsBehavior` or `ImagePlaygroundViewController` do not send data to Apple.

#### Mistral / Groq / Cohere / Perplexity (Third-Party LLM APIs)
| Data Type | Linked | Tracking | Purpose | Notes |
|-----------|--------|----------|---------|-------|
| User Content (prompts/messages) | ASK | No | App Functionality | Sent to third-party AI server |

> Treat any third-party LLM API the same as OpenAI: declare User Content as collected data, disclose the provider, obtain explicit user consent. Check each provider's specific data retention and training policies.

---

### Phase 5: Interactive Disambiguation

After completing the automated scan, first answer the App Store Connect gate question, then ask disambiguation questions for any ambiguous cases.

#### App Store Connect Gate Question (always answer this first)

**0. Data Collection**: "Does your app or any third-party SDK transmit data off the device for any purpose (advertising, analytics, purchases, crash reporting, etc.)?"

- If **ANY** privacy-relevant SDK was detected (AdMob, RevenueCat, Firebase Analytics, Crashlytics, attribution SDKs, etc.) → the answer is **"Yes, we collect data from this app"**
- Only if NO SDK collects ANY data AND all storage is strictly on-device → answer "No, we do not collect data from this app"

This gate answer must appear at the top of the App Store Connect checklist in the report (Output 2).

#### Disambiguation Questions for Ambiguous Cases

1. **Firestore/Cloud Storage**: "I detected Firestore usage. What user data do you store? (e.g., profile info with name/email, health records, usage logs, custom content)"
2. **Location**: "I detected CoreLocation. Is location data: (a) used only on-device, (b) sent to your server, (c) shared with third parties?"
3. **Tracking**: "I detected ATTrackingManager. Do you share data with third-party ad networks or use IDFA for cross-app tracking?"
4. **Camera/Photos**: "I detected camera/photo library access. Are captured photos/videos: (a) stored on-device only, (b) uploaded to your server, (c) shared with third parties?"
5. **Contacts**: "I detected Contacts framework usage. Are contacts: (a) used on-device only, (b) uploaded to your server?"
6. **Custom APIs**: "I detected network requests to your own API endpoints. What user data is transmitted? (e.g., name, email, health data, usage metrics, location)"
7. **Analytics user linking**: "I detected Firebase Analytics. Do you call `Analytics.setUserID()` to link analytics to user identity?"
8. **Data purposes**: For each detected data type, confirm: "Is [data type] used for: App Functionality / Analytics / Personalization / Advertising / Other?"

Wait for the user's answers before proceeding to Phase 6.

---

### Phase 6: PrivacyInfo.xcprivacy Generation

Generate a `PrivacyInfo.xcprivacy` file for **each platform target** detected in Phase 1.

#### Multi-Platform Handling

1. **Identify all targets** from the Xcode project (e.g., `MyApp` for iOS, `MyApp Watch App` for watchOS, `MyApp Mac` for macOS)
2. **Generate a separate `PrivacyInfo.xcprivacy`** per target, because each platform target may:
   - Use different SDKs (e.g., AdMob on iOS but not on watchOS)
   - Access different APIs (e.g., HealthKit on watchOS and visionOS, ARKit on visionOS)
   - Have different data collection patterns due to `#if os()` conditional compilation
3. **Place each file** in the corresponding target directory:
   - `MyApp/PrivacyInfo.xcprivacy` (iOS/iPadOS)
   - `MyApp Watch App/PrivacyInfo.xcprivacy` (watchOS)
   - `MyApp Mac/PrivacyInfo.xcprivacy` (macOS)
   - `MyApp Vision/PrivacyInfo.xcprivacy` (visionOS)
   - `MyApp TV/PrivacyInfo.xcprivacy` (tvOS)
   - If target directories cannot be determined, place all files in the project root with platform suffixes: `PrivacyInfo-iOS.xcprivacy`, `PrivacyInfo-watchOS.xcprivacy`, etc.
4. **If the project has only one target**, generate a single `PrivacyInfo.xcprivacy` in the project root

> **Important**: Only include data types and APIs relevant to each specific platform target. For example, a watchOS target that only uses HealthKit should not include AdMob-related declarations that exist only in the iOS target.

#### PrivacyInfo.xcprivacy Format

Generate a valid Apple privacy manifest plist file. Use the exact Apple constant names.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <!-- One dict per collected data type -->
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>[constant]</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/> or <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <true/> or <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>[purpose constant]</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <!-- One dict per Required Reason API used -->
    </array>
</dict>
</plist>
```

Set `NSPrivacyTracking` to `<true/>` only if the app uses data for tracking (ATT required).

For `NSPrivacyTrackingDomains`, include domains of third-party trackers if tracking is enabled (e.g., `analytics.google.com`, `graph.facebook.com`).

---

### Phase 7: Age Rating Questionnaire

After generating the privacy report, analyze the codebase to answer Apple's App Store Connect Age Rating questionnaire. Scan code patterns and detected SDKs to determine answers.

#### Age Rating Content Descriptors

For each descriptor, determine the frequency: **None**, **Infrequent/Mild**, or **Frequent/Intense**.

##### Violence
| Descriptor | How to Detect |
|-----------|---------------|
| Cartoon or Fantasy Violence | Grep for game-related violence: `SpriteKit`, `SceneKit` game scenes, explosion/combat animations, `SKAction`, game physics with destruction. Check app category — games are more likely. |
| Realistic Violence | Grep for realistic combat, weapon models, blood/gore assets, violent sound effects. Check asset catalogs for violent imagery. |
| Prolonged Graphic or Sadistic Realistic Violence | Extremely rare in standard apps. Flag only if explicit violent content systems are detected. |
| Guns or Other Weapons | Grep for weapon-related assets, models (`.usdz`, `.reality`), or references in code (`weapon`, `gun`, `sword`, `knife`). |

##### Profanity & Mature Themes
| Descriptor | How to Detect |
|-----------|---------------|
| Profanity or Crude Humor | Grep for profanity filter/word list implementations, check localization files for crude content. If app has UGC (user-generated content), flag as potential. |
| Horror/Fear Themes | Grep for horror-related assets, dark/scary themes, jump scare logic, creepy sound effects. Check app category. |
| Mature or Suggestive Themes | Check for dating features, mature content flags, age-gated sections. |

##### Substances
| Descriptor | How to Detect |
|-----------|---------------|
| Alcohol, Tobacco, or Drug Use or References | Grep for substance-related content, check asset names, localization strings for references to alcohol/drugs/smoking. |

##### Sexuality
| Descriptor | How to Detect |
|-----------|---------------|
| Sexual Content or Nudity | Grep for NSFW filters, adult content flags, nudity detection. Typically None for standard apps. |
| Graphic Sexual Content and Nudity | Almost always None. Flag only if explicit adult content systems detected. Results in **Unrated** (cannot be published). |

##### Medical
| Descriptor | How to Detect |
|-----------|---------------|
| Medical or Treatment Information | Grep for `HealthKit`, medical terminology in strings, diagnosis/treatment content, medication databases. If HealthKit detected → at minimum **Infrequent**. |
| Health or Wellness Topics | Grep for fitness tracking, mental health features, wellness content, mindfulness, nutrition. If fitness/health features detected → at minimum **Infrequent**. |

#### Age Rating Boolean Questions (Yes/No)

##### Capabilities
| Question | How to Detect | Default |
|----------|---------------|---------|
| **Unrestricted Web Access** | Grep for `WKWebView`, `SFSafariViewController` loading arbitrary URLs (not just specific known URLs). `SFSafariViewController` with hardcoded URLs = No. `WKWebView` loading user-input URLs = Yes. | No |
| **User-Generated Content** | Grep for text input fields that post to a shared server, photo/video upload to shared feeds, comments, reviews, social features, chat. Check for `Firestore` writes that are visible to other users. | No |
| **Messaging and Chat** | Grep for real-time messaging, chat UI (`MessageKit`, custom chat), WebSocket connections for chat, `Stream`, `SendBird`, `Firebase Realtime Database` for chat. | No |
| **Advertising** | If `GoogleMobileAds`, `AdMob`, `GADBannerView`, `GADInterstitialAd`, `GADRewardedAd`, Facebook Ads, or any ad SDK detected → **Yes**. | No |

##### Controls
| Question | How to Detect | Default |
|----------|---------------|---------|
| **Parental Controls** | Grep for parental gate, age verification, parental PIN, `DeviceCheck`, Screen Time API integration. | No |
| **Age Assurance** | Grep for age verification flows, date of birth collection for age gating. | No |

##### Chance-Based Activities
| Question | How to Detect | Default |
|----------|---------------|---------|
| **Gambling** (real money) | Grep for real-money wagering, casino, betting APIs. Almost always None for standard apps. | None |
| **Simulated Gambling** | Grep for slot machine UI, casino-style games, poker, roulette without real money. | None |
| **Contests** | Grep for contest/sweepstakes/giveaway features, competition entry. If detected → Infrequent or Frequent based on prominence. | None |
| **Loot Boxes** | Grep for randomized reward systems, gacha mechanics, mystery boxes, random item drops tied to purchases. | None |

#### Age Rating Mapping Table

| Rating | Triggers |
|--------|----------|
| **4+** | None/minimal content. May include: parental controls, UGC, messaging, advertising, infrequent contests |
| **9+** | Infrequent: profanity, horror, suggestive themes, cartoon violence. Health/wellness topics. Loot boxes. |
| **12+** | Frequent: profanity, horror, cartoon violence. Infrequent: drugs, sexual content, realistic violence, simulated gambling. Frequent contests. |
| **17+** | Frequent: drugs/alcohol, sexual content, realistic violence, simulated gambling. Unrestricted web access. Frequent medical info. |
| **Unrated** | Graphic sexual content/nudity OR prolonged graphic violence → **Cannot be published** |

#### Age Rating Output Format

Include in the report:

```markdown
## Age Rating Questionnaire

**Recommended Rating: [4+ / 9+ / 12+ / 17+]**

### Violence
| Descriptor | Answer | Reason |
|-----------|--------|--------|
| Cartoon or Fantasy Violence | None | No game or violence-related code detected |
| Realistic Violence | None | No violent content detected |
| Prolonged Graphic or Sadistic Realistic Violence | None | N/A |
| Guns or Other Weapons | None | No weapon references detected |

### Profanity & Mature Themes
| Descriptor | Answer | Reason |
|-----------|--------|--------|
| Profanity or Crude Humor | None | No profanity content detected |
| Horror/Fear Themes | None | No horror content detected |
| Mature or Suggestive Themes | None | No mature themes detected |

### Substances
| Descriptor | Answer | Reason |
|-----------|--------|--------|
| Alcohol, Tobacco, or Drug Use or References | None | No substance references detected |

### Sexuality
| Descriptor | Answer | Reason |
|-----------|--------|--------|
| Sexual Content or Nudity | None | No sexual content detected |
| Graphic Sexual Content and Nudity | None | N/A |

### Medical
| Descriptor | Answer | Reason |
|-----------|--------|--------|
| Medical or Treatment Information | [answer] | [e.g., "HealthKit detected — app provides health data"] |
| Health or Wellness Topics | [answer] | [reason] |

### Capabilities (Yes/No)
| Question | Answer | Reason |
|----------|--------|--------|
| Unrestricted Web Access | [Yes/No] | [reason] |
| User-Generated Content | [Yes/No] | [reason] |
| Messaging and Chat | [Yes/No] | [reason] |
| Advertising | [Yes/No] | [e.g., "AdMob SDK detected"] |

### Controls (Yes/No)
| Question | Answer | Reason |
|----------|--------|--------|
| Parental Controls | [Yes/No] | [reason] |
| Age Assurance | [Yes/No] | [reason] |

### Chance-Based Activities
| Question | Answer | Reason |
|----------|--------|--------|
| Gambling | None | [reason] |
| Simulated Gambling | None | [reason] |
| Contests | [answer] | [reason] |
| Loot Boxes | [answer] | [reason] |

### Important Notes
- [Any edge cases or items requiring manual verification]
- [e.g., "App has UGC — ensure content moderation is in place"]
```

> **Bias toward higher rating**: When unsure, recommend the higher rating. A rejected submission for incorrect age rating is worse than a slightly conservative rating.

---

### Phase 8: AI Privacy Disclosure

If ANY AI SDK or endpoint was detected in Phases 2 or 3, generate a complete **AI Data Practices** section and a standalone `Docs/AI-PRIVACY-DISCLOSURE.md` file.

#### 8.0 — AI Provider Detection Summary

Build a table of all AI providers found:

| Provider | SDK / Pattern Detected | Processing Location | Data Sent | Trains on Data | Retention |
|----------|----------------------|-------------------|-----------|----------------|-----------|
| Apple Intelligence (Foundation Models) | `LanguageModelSession` | On-device | Nothing | No | N/A |
| Apple Intelligence + ChatGPT extension | `ImagePlaygroundViewController` + ChatGPT route | On-device + OpenAI (opt-in) | Prompts (user-consented only) | No (OpenAI API policy) | 30 days (OpenAI) |
| OpenAI (GPT-4 / GPT-4o) | `OpenAI(apiKey:` / `api.openai.com` | OpenAI servers (US) | Prompts + context | No (API default) | 30 days |
| Google Gemini (free tier) | `GenerativeModel(` / `generativelanguage.googleapis.com` | Google servers | Prompts + responses | **YES** (free tier) | Unspecified |
| Google Gemini via Firebase AI | `import FirebaseAI` / `firebaseai.googleapis.com` | Google servers | Prompts + responses | No (paid/Firebase tier) | Limited (abuse detection) |
| Anthropic Claude | `Anthropic(apiKey:` / `api.anthropic.com` | Anthropic servers (US) | Prompts + responses | Opt-out available | 30 days |
| Mistral / Groq / Cohere / Other | URL endpoint strings | Third-party servers | Prompts + responses | Check provider | Check provider |
| CoreML / NaturalLanguage (on-device) | `MLModel(` / `NLModel(` | On-device | Nothing | No | N/A |

> Only include rows for providers actually detected. Ask user to confirm free vs paid tier for Gemini.

#### 8.1 — AI Disambiguation Questions

Ask the user the following before generating disclosures:

1. **Data sensitivity**: "What types of user data are included in AI prompts? (e.g. only text the user types, or also name/email/health data/location?)"
2. **Gemini tier** (if detected): "Is this app using the free Gemini API tier (AI Studio) or Firebase AI (formerly Vertex AI for Firebase / paid tier)?"
3. **Anthropic training opt-out** (if Claude detected): "Have you opted out of Anthropic using your API data for model training in your Anthropic account settings?"
4. **OpenAI zero-retention** (if OpenAI detected): "Do you have an OpenAI enterprise agreement with zero data retention?"
5. **User consent flow**: "Does the app show users a disclosure before their first AI interaction explaining that prompts are sent to [provider]?"

#### 8.2 — App Store Connect AI Declarations (§5.1.2(i))

Per **§5.1.2(i)**: "You must clearly disclose where personal data will be shared with third parties, **including with third-party AI**, and obtain explicit permission before doing so."

For each cloud AI provider detected, declare in App Store Connect:

| Provider | Data Type to Declare | NSPrivacyCollectedDataType Constant | Linked | Tracking | Purpose |
|----------|---------------------|-------------------------------------|--------|----------|---------|
| OpenAI | User Content (prompts) | `NSPrivacyCollectedDataTypeOtherUserContent` | ASK | No | App Functionality |
| OpenAI | Other Data (AI responses cached/stored) | `NSPrivacyCollectedDataTypeOtherDataTypes` | No | No | App Functionality |
| Google Gemini (free tier) | User Content | `NSPrivacyCollectedDataTypeOtherUserContent` | No | No | App Functionality |
| Google Gemini (free tier) | Product Interaction (used for training) | `NSPrivacyCollectedDataTypeProductInteraction` | No | No | Analytics |
| Anthropic Claude | User Content | `NSPrivacyCollectedDataTypeOtherUserContent` | ASK | No | App Functionality |
| Any cloud AI + user-identifiable prompts | Name, Email, Health Data, etc. | (relevant constant) | Yes | No | App Functionality |

> **Apple Intelligence (on-device only)**: No App Store Connect declaration needed. Processing never leaves the device.

#### 8.3 — NSPrivacyAccessedAPITypes additions for AI

If AI features use UserDefaults (to store conversation history, AI preferences, model selection), ensure `NSPrivacyAccessedAPICategoryUserDefaults` with reason `CA92.1` is in the `PrivacyInfo.xcprivacy`.

#### 8.4 — Required In-App AI Disclosures

Generate the recommended user-facing disclosure text for each provider. This text must appear in the app's **Privacy Policy** and optionally as an **in-app disclosure** shown before the first AI interaction.

---

**Apple Intelligence (Foundation Models)**
```
This app uses Apple Intelligence, which processes your requests using an
on-device AI model. Your data never leaves your device.
```

---

**OpenAI (GPT-4 / GPT-4o)**
```
This app uses OpenAI's GPT models to [describe feature, e.g. "generate
responses"]. When you use [feature name], your messages are sent to OpenAI's
servers and processed in the United States. OpenAI retains API data for up
to 30 days for safety monitoring. Your data is not used to train OpenAI's
models. For more information, see OpenAI's Privacy Policy at openai.com/privacy.
```

---

**Google Gemini (Free Tier)**
```
This app uses Google Gemini to [describe feature]. When you use [feature name],
your messages are sent to Google's servers. Google may use this data to improve
its AI models and services. Human reviewers at Google may read content in
de-identified form. For more information, see Google's Privacy Policy.

[WARN: Free tier requires prominent disclosure — consider upgrading to paid tier
to eliminate training data concerns.]
```

---

**Google Gemini (Paid Tier)**
```
This app uses Google Gemini to [describe feature]. When you use [feature name],
your messages are sent to Google's servers. Google does not use API data to
train its models. Data is retained for a limited period for safety monitoring only.
```

---

**Anthropic Claude**
```
This app uses Anthropic's Claude AI to [describe feature]. When you use
[feature name], your messages are sent to Anthropic's servers and processed
in the United States. Anthropic retains data for up to 30 days. [If opted out:
Your data is not used to train Claude's models.] For more information, see
Anthropic's Privacy Policy at anthropic.com/privacy.
```

---

**Apple Intelligence + ChatGPT Extension (opt-in)**
```
Some features use Apple Intelligence, which processes your requests on-device.
When you choose to use ChatGPT-enhanced features, your request is sent to
OpenAI. Apple obscures your IP address before sending. You will always be asked
before any data is sent to ChatGPT.
```

---

#### 8.5 — AI Privacy Compliance Checks

| Check | Guideline | Status | Finding |
|-------|-----------|--------|---------|
| Third-party AI disclosure in app/privacy policy | §5.1.2(i) | 🚨/✅ | "Prompts sent to [provider] — must disclose" |
| User consent before first AI use | §5.1.2(i) | 🚨/✅ | "Explicit consent required before sending data to AI" |
| Gemini free tier training disclosure | §5.1.2(i) | ⚠️/✅ | "Free tier may train on data — disclose or upgrade" |
| HealthKit / sensitive data in AI prompts | §5.1.2(vi) | 🚨/✅ | "Sensitive data in prompts may not go to third-party AI without consent" |
| Age rating for AI chatbot features | §4.7 / §4.7.5 | ⚠️/✅ | "Chatbot features require age restriction mechanisms" |
| AI content moderation for UGC | §4.7.1 | ⚠️/✅ | "AI-powered UGC requires content filtering and abuse reporting" |

#### 8.6 — Generate AI-PRIVACY-DISCLOSURE.md

##### Step 1 — Detect App Theme & Branding

Before asking the user, scan the codebase for visual identity:

1. **Accent color** — Grep `Assets.xcassets` for a `AccentColor.colorset` or `AppTint.colorset`. Read the `.colorset`'s `Contents.json` to extract the hex/rgb value.
2. **Brand colors** — Grep for `Color(hex:`, `UIColor(red:`, `Color("`, or any `extension Color` / `extension ShapeStyle` defining custom app colors. Note the top 2–3 values used most frequently.
3. **App name** — Read from `Info.plist` (`CFBundleDisplayName` or `CFBundleName`) or from the Xcode project target name.
4. **App icon** — Check `Assets.xcassets/AppIcon.appiconset/` for the largest available icon file.
5. **Design style** — Grep for `.rounded`, `.ultraThinMaterial`, SF Symbol usage, `GradientBackground`, and note whether the app uses a **light/minimalist**, **dark**, or **colorful/vibrant** design language.

Build a theme profile:
```
App Name:       [name]
Primary Color:  #[hex] (or "system default" if not found)
Secondary Color: #[hex] (or none)
Design Style:   Light / Dark / Vibrant / Minimal
Icon:           [path or N/A]
```

##### Step 2 — Ask User

> "I detected the following AI providers in your app: **[list providers]**.
>
> I also detected your app's branding:
> - App name: **[App Name]**
> - Primary color: **[#hex or "system accent"]**
> - Design style: **[Light / Dark / Vibrant]**
>
> Would you like me to generate a beautifully styled `AI-PRIVACY-DISCLOSURE.md` using your app's theme? It will include:
> - In-app disclosure text per AI feature (copy-paste ready)
> - Privacy policy language per provider
> - App Store Connect declarations checklist
> - Compliance status per §5.1.2(i)
>
> Reply **Yes** to generate with your app's branding, or **No** to skip."

Only generate if the user confirms **Yes**.

##### Step 3 — Generate with App Theme

Use the **`canvas-design` skill** to generate a beautifully designed PDF version at `Docs/AI-PRIVACY-DISCLOSURE.pdf`, styled with the detected app colors, app name, and icon. Then also generate the plain `Docs/AI-PRIVACY-DISCLOSURE.md` as a developer-readable copy.

Apply the detected theme to the canvas-design PDF:
- **Header**: App icon + App name + "AI Data Practices" subtitle, using primary brand color
- **Section headers**: Primary accent color
- **Tables**: Light/dark tinted rows matching the app's design style
- **Footer**: "Generated by App Privacy Skill · [date]" in secondary color or gray
- **Typography**: SF Pro Display for headings (or system equivalent), clean readable body

If `canvas-design` skill is not available, generate `Docs/AI-PRIVACY-DISCLOSURE.md` only (plain markdown), with a comment block at the top noting the detected app theme:

```markdown
<!-- App Theme: [App Name] | Primary: #[hex] | Style: [Light/Dark/Vibrant] -->
```

Create `Docs/AI-PRIVACY-DISCLOSURE.md` with the following structure:

```markdown
# AI Data Practices — [App Name]
Generated: [date]

## AI Features Overview

| Feature | AI Provider | Processing Location | User Data Sent |
|---------|------------|-------------------|----------------|
| [feature] | [provider] | On-device / Cloud | [data types] |

## Provider-by-Provider Data Practices

### [Provider Name]
- **Data sent**: [description]
- **Server location**: [country/region]
- **Trains on your data**: Yes / No / Opt-out available
- **Retention period**: [X days / limited / none]
- **Zero retention available**: Yes / No
- **Apple §5.1.2(i) disclosure required**: Yes / No

## App Store Connect Declarations Required
[table from 8.2]

## Recommended Privacy Policy Language
[generated text from 8.4]

## Recommended In-App Disclosure
[short consent text to show users before first AI use]

## Compliance Status
[table from 8.5]
```

---

### Phase 9: App Store Compliance Checks

After Age Rating analysis and before generating the report, run these submission compliance checks. These are derived from Apple Review Guidelines and flag critical rejection risks found in greenlight-style scanning.

#### 8a — ATT Cross-Check (§5.1.2)

If **any** of these were detected in Phase 2 or 3:
- `GoogleMobileAds`, `GADBannerView`, `GADInterstitialAd`, `GADRewardedAd`
- `FacebookCore`, `FBSDKCoreKit`
- `AdSupport`, `ASIdentifierManager`, `advertisingIdentifier`
- `Adjust`, `AppsFlyer`, `Branch`, `Singular`, `Kochava`

Then **verify ATT is implemented** by grepping for:
- `ATTrackingManager.requestTrackingAuthorization`
- `import AppTrackingTransparency`

| Result | Severity | Finding |
|--------|----------|---------|
| Tracking SDK found + ATT missing | **CRITICAL** | App will be rejected — ATT is mandatory for tracking (§5.1.2) |
| Tracking SDK found + ATT present | ✅ PASS | ATT correctly implemented |
| No tracking SDK | ✅ N/A | ATT not required |

#### 8b — Info.plist Purpose String Quality (§5.1.1)

Grep for all `NSUsageDescription` keys in `Info.plist` files. Flag any string that:
- Is empty or missing when a permission is requested
- Is generic: "Used by the app", "Required", "Camera needed", "Location access", "For app functionality"
- Does not mention the specific feature that uses the data

| Severity | Example |
|----------|---------|
| **CRITICAL** | Key exists but value is empty |
| **WARN** | Value is vague — "Camera needed" |
| **INFO** | Value could be more specific |

**Expected keys to check against detected permissions:**
- `NSCameraUsageDescription` — if `AVCaptureSession` or `AVCaptureDevice` detected
- `NSMicrophoneUsageDescription` — if `AVAudioSession` or `AVCaptureDevice` for audio
- `NSPhotoLibraryUsageDescription` — if `PHPhotoLibrary` or `PHPickerViewController`
- `NSLocationWhenInUseUsageDescription` — if `CLLocationManager`
- `NSLocationAlwaysAndWhenInUseUsageDescription` — if `requestAlwaysAuthorization`
- `NSContactsUsageDescription` — if `CNContactStore`
- `NSHealthShareUsageDescription` — if `HKHealthStore`
- `NSHealthUpdateUsageDescription` — if `HKHealthStore` with write access
- `NSMotionUsageDescription` — if `CMMotionManager`
- `NSSpeechRecognitionUsageDescription` — if `SFSpeechRecognizer`
- `NSFaceIDUsageDescription` — if `LocalAuthentication`

#### 8c — Account Deletion Check (§5.1.1)

If **any account creation** was detected (email sign-up, Google Sign In, Apple Sign In, Firebase Auth), then grep for account deletion:
- `deleteUser()`, `delete(completion:)` on Firebase Auth user
- Strings: `"delete account"`, `"deleteAccount"`, `"Delete Account"` in localizable strings or Swift code
- Navigation to a settings/account deletion screen

| Result | Severity | Finding |
|--------|----------|---------|
| Auth detected + no deletion found | **WARN** | Account deletion is required by §5.1.1. Add a "Delete Account" option in settings. |
| Auth detected + deletion found | ✅ PASS | Account deletion is implemented |
| No auth detected | ✅ N/A | Not required |

#### 8d — Restore Purchases Check (§3.1.1)

If **StoreKit** (`SKPaymentQueue`, `Product.products`, `Transaction.currentEntitlements`) was detected, grep for restore purchases:
- `SKPaymentQueue.default().restoreCompletedTransactions()`
- `Transaction.currentEntitlements`
- `AppStore.sync()`
- Strings: `"restore"`, `"restorePurchases"`, `"Restore Purchases"`

| Result | Severity | Finding |
|--------|----------|---------|
| IAP detected + no restore found | **WARN** | Restore Purchases is required for all IAP apps (§3.1.1) |
| IAP detected + restore found | ✅ PASS | Restore is implemented |
| No IAP detected | ✅ N/A | Not required |

#### 8e — External Payment for Digital Goods (§3.1.1)

Grep for external payment processors being used:
- `import Stripe`, `STPPaymentContext`, `STPPaymentHandler`
- `import PayPal`, `PayPalCheckout`
- `import Braintree`
- `checkout.stripe.com`, `paypal.com/checkout` in string literals

If detected, check if the app's context is **physical goods** (food delivery, e-commerce, ride sharing). If it's a digital product app:

| Result | Severity | Finding |
|--------|----------|---------|
| External payment + digital goods context | **CRITICAL** | External payment for digital goods violates §3.1.1. Use StoreKit/IAP instead. |
| External payment + physical goods context | **INFO** | Allowed for physical goods — verify context is correct. |

> Ask the user: "I detected Stripe/PayPal. Does this app sell physical goods (allowed) or digital content/subscriptions (must use StoreKit)?"

#### 8f — Sign in with Apple Parity (§4.8)

If **any** of these social logins are detected:
- `GIDSignIn` (Google)
- `FacebookCore`, `FBSDKCoreKit` (Facebook)
- `OAuthProvider` for any provider

Then verify **Sign in with Apple** is also present:
- `ASAuthorizationAppleIDProvider`
- `import AuthenticationServices`

| Result | Severity | Finding |
|--------|----------|---------|
| Social login found + no Apple Sign In | **CRITICAL** | Sign in with Apple is required when offering social login (§4.8) |
| Social login found + Apple Sign In present | ✅ PASS | Apple Sign In parity satisfied |
| No social login | ✅ N/A | Not required |

#### Compliance Summary Output

Include a **Compliance Findings** section in the report with this table:

```
## App Store Compliance Findings

| Check | Guideline | Status | Finding |
|-------|-----------|--------|---------|
| ATT for tracking SDKs | §5.1.2 | ✅ PASS / ⚠️ WARN / 🚨 CRITICAL | [reason] |
| Purpose string quality | §5.1.1 | ... | [reason] |
| Account deletion | §5.1.1 | ... | [reason] |
| Restore Purchases | §3.1.1 | ... | [reason] |
| External payment | §3.1.1 | ... | [reason] |
| Apple Sign In parity | §4.8 | ... | [reason] |
| Third-party AI disclosure | §5.1.2(i) | ... | [e.g. "OpenAI prompts — disclose in privacy policy"] |
| Gemini free tier training | §5.1.2(i) | ... | [e.g. "Free tier trains on data — disclose or upgrade to paid"] |
| Chatbot age restriction | §4.7.5 | ... | [e.g. "Chat feature requires age-restriction mechanism"] |
```

🚨 = Must fix before submission | ⚠️ = Should fix | ✅ = No action needed

---

### Phase 9.5: Output Format Choice

Before generating the final report, ask the user:

> **"How would you like the report delivered?"**
> - **A) `.docx` only** — Professional Word document (requires `npm install -g docx`)
> - **B) `.md` only** — Markdown report (no dependencies)
> - **C) Both** — `.docx` + `Docs/APP-PRIVACY-REPORT.md` (default recommendation)
>
> *(If no answer or default accepted, generate both.)*

Store the choice and branch Phase 10 accordingly:
- Choice A or C → run Phase 10 docx generation
- Choice B or C → write `Docs/APP-PRIVACY-REPORT.md` (8-section structure: Executive Summary, Detected SDKs & Frameworks, Privacy Declaration, Required Reason APIs, Per-Target Declaration Summary, App Store Connect Checklist, Age Rating, v2/Future Notes)

---

### Phase 10: Document Generation (.docx Report)

After generating the `PrivacyInfo.xcprivacy` and completing the Age Rating analysis, create a professional `.docx` document containing the complete report. This is the **primary output** of the skill.

**IMPORTANT**: Use the `docx` skill to generate the Word document. The docx skill must be available (installed at `~/.claude/skills/docx/` or in the project's `.claude/skills/docx/`).

#### Document Structure

Generate a `.docx` file named `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` saved to the `Docs/` folder in the project root (create the folder if it doesn't exist). This keeps a versioned copy alongside the project for future reference.

```bash
mkdir -p Docs/
```

The document has the following structure:

1. **Title Page**
   - App Name (large, bold)
   - "App Store Privacy & Age Rating Report"
   - Generated date
   - Target platform(s)

2. **Table of Contents**

3. **Executive Summary**
   - Data collection gate answer (Yes/No with reason)
   - Data collection summary table (counts by linkage category: Track / Linked / Not Linked / Total)
   - Recommended age rating
   - Key findings (detected SDKs count, tracking status)

4. **Detected SDKs & Frameworks**
   - Table with columns: SDK/Framework | Detected In (file:line) | Privacy Impact | Confidence (HIGH/MEDIUM/LOW)

5. **Privacy Declaration by Data Category**
   - One subsection per Apple data category (all 14):
     Contact Info, Health & Fitness, Financial Info, Location, Sensitive Info, Contacts, User Content, Browsing History, Search History, Identifiers, Purchases, Usage Data, Diagnostics, Surroundings, Body, Other Data
   - Each with table: Data Type | Collected | Linked to You | Used to Track | Purpose(s) | Detected Source
   - Skip categories with no data detected (note "No data collected in this category")

6. **App Store Connect Checklist**
   - Step 1: Data Collection Gate answer with reason
   - Step 2: Data types organized by:
     - Data Used to Track You (checklist with checkboxes)
     - Data Linked to You (checklist with checkboxes)
     - Data Not Linked to You (checklist with checkboxes)
   - Each item: ☑ Data Type — Purpose: [purpose] — Source: [SDK]

7. **Age Rating Questionnaire**
   - Recommended rating (highlighted/bold)
   - Violence descriptors table (Descriptor | Answer | Reason)
   - Profanity & Mature Themes table
   - Substances table
   - Sexuality table
   - Medical table
   - Capabilities table (Yes/No answers with reasons)
   - Controls table
   - Chance-Based Activities table
   - Important notes for edge cases

8. **AI Data Practices** *(only if AI SDKs detected)*
   - AI features overview table (Feature | Provider | Processing | Data sent)
   - Per-provider data practices (retention, training, server location)
   - App Store Connect declarations required for AI data
   - Recommended privacy policy language per provider
   - Recommended in-app consent disclosure text
   - AI compliance findings (§5.1.2(i), §4.7, §4.7.5)

9. **App Store Compliance Findings**
   - Summary table: Check | Guideline | Status | Finding
   - All compliance checks including AI-specific (§5.1.2(i), Gemini training, chatbot age gates)
   - Color-coded: 🚨 CRITICAL / ⚠️ WARN / ✅ PASS

10. **PrivacyInfo.xcprivacy Contents**
   - One subsection per platform target (e.g., "iOS Target", "watchOS Target")
   - The full XML content for each target formatted as a code block (monospace font)

11. **Exemptions Applied**
    - List of exempted data types with reasoning

12. **Confidence Notes & Limitations**
    - Confidence level definitions (HIGH / MEDIUM / LOW)
    - Items requiring manual verification
    - Known limitations

#### Document Formatting

- Use professional styling: Heading 1 for sections, Heading 2 for subsections
- Tables with header row shading (light blue or gray)
- Checklist items as bullet points with checkbox characters (☐ / ☑)
- Code blocks for XML content (monospace font)
- Color coding for confidence: GREEN = HIGH, ORANGE = MEDIUM, RED = LOW
- Footer with "Generated by App Privacy Skill — [date]"

#### How to Generate

Use the battle-tested `docx-js` pattern below. Follow every step exactly.

**Step 1 — Install docx npm package**

```bash
npm install -g docx 2>&1 | tail -3
```

**Step 2 — Resolve NODE_PATH (global modules may not be on the default path)**

```bash
NODE_PATH=$(npm root -g)
```

**Step 3 — Write the generation script to `/tmp/gen_privacy_report.js`**

Use the Write tool to create the Node.js script. The script must:
- Require docx from the resolved NODE_PATH: `require('${NODE_PATH}/docx')` (template literal evaluated at runtime)
- Use US Letter page size: `width: 12240, height: 15840` (DXA units)
- Set 1" margins: `top: 1440, right: 1440, bottom: 1440, left: 1440`
- Use `LevelFormat.BULLET` for checkbox and bullet lists (never unicode bullets inline)
- Use `ShadingType.CLEAR` for table cell shading (never `SOLID` — breaks rendering)
- Set dual widths on all tables: `columnWidths` array on the table + cell-level `width` (both must sum correctly to total table width)
- Use `WidthType.DXA` exclusively for all widths (never `PERCENTAGE` — breaks Google Docs)
- Include branded header (report title + date) and footer (page numbers)
- Ensure `Docs/` directory exists before writing: `fs.mkdirSync('Docs', { recursive: true })`
- Write output to `Docs/APP-PRIVACY-AND-AGE-RATING-REPORT.docx`
- Exit with code 1 on any error (`process.exit(1)`)

**Step 4 — Run the script**

```bash
NODE_PATH=$(npm root -g) node /tmp/gen_privacy_report.js
```

**Step 5 — Validate (if validator available)**

Check for validator in this order:
1. `~/.claude/skills/docx/scripts/office/validate.py` (global docx skill install)
2. `scripts/office/validate.py` (local copy in skill dir)

```bash
python3 ~/.claude/skills/docx/scripts/office/validate.py \
  Docs/APP-PRIVACY-AND-AGE-RATING-REPORT.docx
```

If the validator is not found at either path, skip silently and note: "Validation skipped — validate.py not found."

**Fallback (if docx install fails)**

If `npm install -g docx` fails, fall back to `.md` only and inform the user:
> "Could not install the `docx` npm package. Generating markdown report only. To get the `.docx` report, run `npm install -g docx` and re-run the skill."

---

## Apple Privacy Constants Reference

### Collected Data Type Constants (`NSPrivacyCollectedDataType`)

| Apple Constant | Data Type |
|----------------|-----------|
| `NSPrivacyCollectedDataTypeName` | Name |
| `NSPrivacyCollectedDataTypeEmailAddress` | Email Address |
| `NSPrivacyCollectedDataTypePhoneNumber` | Phone Number |
| `NSPrivacyCollectedDataTypePhysicalAddress` | Physical Address |
| `NSPrivacyCollectedDataTypeOtherContactInfo` | Other Contact Info |
| `NSPrivacyCollectedDataTypeHealthData` | Health Data |
| `NSPrivacyCollectedDataTypeFitnessData` | Fitness Data |
| `NSPrivacyCollectedDataTypePaymentInfo` | Payment Info |
| `NSPrivacyCollectedDataTypeCreditScore` | Credit Score |
| `NSPrivacyCollectedDataTypeOtherFinancialInfo` | Other Financial Info |
| `NSPrivacyCollectedDataTypePreciseLocation` | Precise Location |
| `NSPrivacyCollectedDataTypeCoarseLocation` | Coarse Location |
| `NSPrivacyCollectedDataTypeSensitiveInfo` | Sensitive Info |
| `NSPrivacyCollectedDataTypeContacts` | Contacts |
| `NSPrivacyCollectedDataTypeEmailsOrTextMessages` | Emails or Text Messages |
| `NSPrivacyCollectedDataTypePhotosOrVideos` | Photos or Videos |
| `NSPrivacyCollectedDataTypeAudioData` | Audio Data |
| `NSPrivacyCollectedDataTypeGameplayContent` | Gameplay Content |
| `NSPrivacyCollectedDataTypeCustomerSupportData` | Customer Support Data |
| `NSPrivacyCollectedDataTypeOtherUserContent` | Other User Content |
| `NSPrivacyCollectedDataTypeBrowsingHistory` | Browsing History |
| `NSPrivacyCollectedDataTypeSearchHistory` | Search History |
| `NSPrivacyCollectedDataTypeUserID` | User ID |
| `NSPrivacyCollectedDataTypeDeviceID` | Device ID |
| `NSPrivacyCollectedDataTypePurchaseHistory` | Purchase History |
| `NSPrivacyCollectedDataTypeProductInteraction` | Product Interaction |
| `NSPrivacyCollectedDataTypeAdvertisingData` | Advertising Data |
| `NSPrivacyCollectedDataTypeOtherUsageData` | Other Usage Data |
| `NSPrivacyCollectedDataTypeCrashData` | Crash Data |
| `NSPrivacyCollectedDataTypePerformanceData` | Performance Data |
| `NSPrivacyCollectedDataTypeOtherDiagnosticData` | Other Diagnostic Data |
| `NSPrivacyCollectedDataTypeEnvironmentScanning` | Environment Scanning |
| `NSPrivacyCollectedDataTypeHandsData` | Hands |
| `NSPrivacyCollectedDataTypeHeadMovement` | Head Movement |
| `NSPrivacyCollectedDataTypeOtherDataTypes` | Other Data Types |

### Purpose Constants (`NSPrivacyCollectedDataTypePurposes`)

| Apple Constant | Purpose |
|----------------|---------|
| `NSPrivacyCollectedDataTypePurposeAppFunctionality` | App Functionality |
| `NSPrivacyCollectedDataTypePurposeAnalytics` | Analytics |
| `NSPrivacyCollectedDataTypePurposeProductPersonalization` | Product Personalization |
| `NSPrivacyCollectedDataTypePurposeDeveloperAdvertising` | Developer's Advertising or Marketing |
| `NSPrivacyCollectedDataTypePurposeThirdPartyAdvertising` | Third-Party Advertising |
| `NSPrivacyCollectedDataTypePurposeOther` | Other Purposes |

### Required Reason API Types (`NSPrivacyAccessedAPITypes`)

| Apple Constant | API |
|----------------|-----|
| `NSPrivacyAccessedAPICategoryFileTimestamp` | File timestamp APIs |
| `NSPrivacyAccessedAPICategorySystemBootTime` | System boot time APIs |
| `NSPrivacyAccessedAPICategoryDiskSpace` | Disk space APIs |
| `NSPrivacyAccessedAPICategoryActiveKeyboards` | Active keyboard APIs |
| `NSPrivacyAccessedAPICategoryUserDefaults` | UserDefaults APIs |

### Required Reason API — Common Reason Codes

#### UserDefaults (`NSPrivacyAccessedAPICategoryUserDefaults`)
| Code | Reason |
|------|--------|
| `CA92.1` | Access user defaults to read/write data accessible only to the app |
| `1C8F.1` | Access user defaults to read/write data accessible to app group |

#### File Timestamp (`NSPrivacyAccessedAPICategoryFileTimestamp`)
| Code | Reason |
|------|--------|
| `C617.1` | Access file timestamps inside the app container |
| `DDA9.1` | Access file timestamps for files the user chose via document picker |
| `3B52.1` | Access file timestamps to determine whether files have changed |

#### System Boot Time (`NSPrivacyAccessedAPICategorySystemBootTime`)
| Code | Reason |
|------|--------|
| `35F9.1` | Access system boot time to calculate elapsed time |

#### Disk Space (`NSPrivacyAccessedAPICategoryDiskSpace`)
| Code | Reason |
|------|--------|
| `E174.1` | Check for sufficient disk space before writing |
| `85F4.1` | Display disk space to the user |

#### Active Keyboards (`NSPrivacyAccessedAPICategoryActiveKeyboards`)
| Code | Reason |
|------|--------|
| `54BD.1` | Custom keyboard app determining active keyboards |

---

## Exemption Rules

Data does NOT need to be declared if ALL of these conditions are true:
1. Not used for tracking or third-party advertising/marketing
2. Collection is infrequent and optional for the user
3. User provides data transparently via the app interface with clear labeling

### Special Exemptions
- **On-device only**: Data processed entirely on-device and never sent off-device does not need to be declared (EXCEPT HealthKit — always declare)
- **Transient data**: Auth tokens, temporary session data, or IP addresses not retained after the request
- **Immediate de-identification**: Data converted immediately (e.g., precise location coarsened before storage)
- **Free-form text**: User-typed text in general text fields where specific data types are not explicitly solicited

### HealthKit Exception
HealthKit data MUST ALWAYS be declared regardless of whether it is processed on-device only. This is an Apple requirement with no exemptions.

---

## Limitations

- **Server-side only**: Cannot detect data collected exclusively on the server (e.g., IP logging, server analytics)
- **Backend linking**: Cannot determine if data is truly "linked to identity" without understanding your backend architecture
- **Custom analytics**: May miss proprietary analytics implementations
- **SDK updates**: Third-party SDK privacy manifests may change with SDK updates — verify against latest SDK documentation
- **WebView content**: Cannot determine what data third-party web content collects within WKWebView
- **Confidence levels**: Detections carry HIGH/MEDIUM/LOW confidence — always review MEDIUM and LOW items manually

---

## Output Files

All output files are written to the project root directory:

| File | Description |
|------|-------------|
| `Docs/APP-PRIVACY-AND-AGE-RATING-REPORT.docx` | Professional Word document — privacy, age rating, AI data practices, compliance findings |
| `Docs/AI-PRIVACY-DISCLOSURE.md` | AI-specific disclosure sheet — only generated if AI SDKs detected |
| `<target>/PrivacyInfo.xcprivacy` | Apple privacy manifest — one per platform target |

If multiple platform targets exist, a separate `PrivacyInfo.xcprivacy` is generated for each with only the data types and APIs relevant to that platform. The `Docs/` folder keeps a versioned history of reports across submissions. The App Store Connect checklist, Age Rating, AI Data Practices, and Compliance Findings are all in the .docx report. `AI-PRIVACY-DISCLOSURE.md` is a standalone shareable sheet developers can paste into their privacy policy.

---

## Important Reminders

1. **Bias toward disclosure**: When in doubt, declare the data type. Over-declaring is safe; under-declaring risks App Store rejection.
2. **Verify with SDK docs**: Always cross-reference the generated report with the latest SDK documentation for each detected framework.
3. **Update on SDK changes**: Re-run this skill whenever you add, remove, or update SDKs/frameworks.
4. **Third-party SDK manifests**: Many SDKs now ship their own `PrivacyInfo.xcprivacy`. This skill focuses on YOUR app's top-level declaration, but remind the user to ensure bundled SDKs include their own manifests.
5. **App Store Connect vs PrivacyInfo.xcprivacy**: These serve different purposes. App Store Connect privacy labels are the user-facing "nutrition labels." PrivacyInfo.xcprivacy is the technical manifest for Required Reason APIs and SDK privacy declarations. Both are needed.
