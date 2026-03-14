---
name: app-privacy
description: Scan SwiftUI/iOS codebases to detect privacy-relevant SDKs, frameworks, and data collection patterns, then generate accurate Apple App Store Privacy Details declarations, Age Rating questionnaire answers, App Store Connect-ready checklists, PrivacyInfo.xcprivacy files, and a professional .docx report document
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

Follow these 8 phases in order. Do NOT skip phases.

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
   - Access different APIs (e.g., HealthKit on watchOS, ARKit on visionOS)
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

### Phase 8: Document Generation (.docx Report)

After generating the `PrivacyInfo.xcprivacy` and completing the Age Rating analysis, create a professional `.docx` document containing the complete report. This is the **primary output** of the skill.

**IMPORTANT**: Use the `docx` skill to generate the Word document. The docx skill must be available (installed at `~/.claude/skills/docx/` or in the project's `.claude/skills/docx/`).

#### Document Structure

Generate a `.docx` file named `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` in the project root with the following structure:

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

8. **PrivacyInfo.xcprivacy Contents**
   - One subsection per platform target (e.g., "iOS Target", "watchOS Target")
   - The full XML content for each target formatted as a code block (monospace font)

9. **Exemptions Applied**
   - List of exempted data types with reasoning

10. **Confidence Notes & Limitations**
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

Use the `docx` skill's `docx-js` approach:

1. Create a Node.js script that uses the `docx` package to build the document programmatically
2. Populate it with all the data gathered in Phases 1-7
3. Save as `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` in the project root
4. Validate the output: `python scripts/office/validate.py APP-PRIVACY-AND-AGE-RATING-REPORT.docx` (if validator available)

If the docx skill is not available, fall back to generating the markdown report only and inform the user that the `.docx` output requires the docx skill to be installed.

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
| `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` | Professional Word document with complete privacy & age rating report |
| `<target>/PrivacyInfo.xcprivacy` | Apple privacy manifest — one per platform target (placed in each target's directory) |

If multiple platform targets exist, a separate `PrivacyInfo.xcprivacy` is generated for each with only the data types and APIs relevant to that platform. The App Store Connect checklist and Age Rating questionnaire are included in the .docx report.

---

## Important Reminders

1. **Bias toward disclosure**: When in doubt, declare the data type. Over-declaring is safe; under-declaring risks App Store rejection.
2. **Verify with SDK docs**: Always cross-reference the generated report with the latest SDK documentation for each detected framework.
3. **Update on SDK changes**: Re-run this skill whenever you add, remove, or update SDKs/frameworks.
4. **Third-party SDK manifests**: Many SDKs now ship their own `PrivacyInfo.xcprivacy`. This skill focuses on YOUR app's top-level declaration, but remind the user to ensure bundled SDKs include their own manifests.
5. **App Store Connect vs PrivacyInfo.xcprivacy**: These serve different purposes. App Store Connect privacy labels are the user-facing "nutrition labels." PrivacyInfo.xcprivacy is the technical manifest for Required Reason APIs and SDK privacy declarations. Both are needed.
