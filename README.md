# App Privacy & Age Rating Skill for Claude Code

Automatically scan your Apple platform Xcode project (iOS, macOS, visionOS, watchOS, tvOS) and generate accurate App Store Privacy Details declarations, Age Rating questionnaire answers, and a professional `.docx` report.

## What It Does

1. **Scans** your project for privacy-relevant SDKs, frameworks, imports, and code patterns
2. **Detects target platforms** (iOS, macOS, visionOS, watchOS, tvOS) and applies platform-specific checks
3. **Maps** findings to Apple's privacy taxonomy (14 data categories, 30+ data types, 6 purposes)
4. **Answers** the Age Rating questionnaire based on detected content (violence, medical, advertising, UGC, etc.)
5. **Asks** clarifying questions for ambiguous cases
6. **Generates** outputs:
   - `PrivacyInfo.xcprivacy` — Valid Apple privacy manifest file (one per platform target)
   - `APP-PRIVACY-AND-AGE-RATING-REPORT.docx` — Professional Word document with full analysis, App Store Connect checklist, and Age Rating answers (requires docx skill)

## Supported Platforms

- iOS / iPadOS
- macOS
- visionOS (with spatial/body tracking detection)
- watchOS (with workout/motion detection)
- tvOS (with multi-user profile detection)

## Supported SDKs & Frameworks

Firebase (Analytics, Auth, Crashlytics, Firestore, Storage, Messaging), RevenueCat, StoreKit, Google Sign-In, Google AdMob, Facebook SDK, HealthKit, CoreLocation, Contacts, ARKit, RealityKit (visionOS), AVFoundation, PhotosUI, Speech, Mixpanel, Amplitude, Segment, Sentry, OneSignal, Braze, Adjust, AppsFlyer, Branch, TelemetryDeck, PostHog, and more.

## Installation

Copy the skill to your Claude Code skills directory:

```bash
# Global installation (available in all projects)
cp -r app-privacy ~/.claude/skills/

# Project-level installation (available only in this project)
cp -r app-privacy /path/to/your/project/.claude/skills/
```

For `.docx` report generation, also install the docx skill:
```bash
cp -r /path/to/skills/docx ~/.claude/skills/
```

## Usage

In Claude Code, say any of:

- "Generate app privacy details for this project"
- "Scan this app for privacy data collection"
- "What privacy data does this app collect?"
- "Help me fill out the App Store privacy section"
- "Generate a PrivacyInfo.xcprivacy for this project"
- "Run a privacy audit"
- "What's the age rating for this app?"
- "App Store submission report"

## Example Output

The skill produces a detailed report organized by Apple's privacy categories:

```
## Summary
| Category                  | Count   |
|---------------------------|---------|
| Data Used to Track You    | 0 types |
| Data Linked to You        | 4 types |
| Data Not Linked to You    | 3 types |
| Total Data Types Collected| 7 types |

## Age Rating: 4+
```

Plus a ready-to-use checklist for App Store Connect, Age Rating questionnaire answers, and a valid `PrivacyInfo.xcprivacy` file.

## Limitations

- Cannot detect server-side-only data collection (e.g., IP logging)
- Cannot determine backend linking without understanding your server architecture
- May miss custom/proprietary analytics implementations
- SDK privacy requirements may change with SDK updates — verify against latest docs
- Does not analyze WKWebView third-party content
- Age rating content detection is best-effort — always verify manually for content-heavy apps

## Re-run When

- You add, remove, or update SDKs/frameworks
- You change what data your app collects or how it's used
- Before every App Store submission

## License

MIT
