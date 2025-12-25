# 🚀 Quick Setup Guide

A quick reference on how to consume `flutter-workflows` in your Flutter projects.

---

## 1. Prerequisites in Your Flutter Project

Make sure your Flutter project has:
- A Git repository hosted on GitHub.
- If using FVM, `.fvm/fvm_config.json` committed.
- Android signing configured in `android/app/build.gradle` (or keystore credentials added as GitHub Secrets).
- For iOS, Fastlane Match and certificates configured in your certificates repository (see [iOS-README.md](iOS-README.md)).

---

## 2. Quick Workflow Examples

### A. Android Release (Build APK/AAB + Upload to Play Store)

Create `.github/workflows/release-android.yml` in your Flutter repository:

```yaml
name: Android Release
on:
  push:
    tags:
      - "*.*.*"

jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-android-build.yml@main
    secrets: inherit
  
  upload:
    needs: build
    if: startsWith(github.ref, 'refs/tags/')
    uses: aquadic/flutter-workflows/.github/workflows/play-store-upload.yml@main
    secrets: inherit
    with:
      play_store_track: 'internal'
```

### B. iOS Release (Build Signed IPA)

Create `.github/workflows/release-ios.yml` in your Flutter repository:

```yaml
name: iOS Release
on:
  push:
    tags:
      - "*.*.*"

jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-ios-build.yml@main
    secrets: inherit
    with:
      runner-type: 'macos-latest'
```

### C. Pull Request Validation (Fast CI)

Create `.github/workflows/pr-validation.yml` in your Flutter repository:

```yaml
name: PR Validation
on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  validate:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-pr-validation.yml@main
    secrets: inherit
```

---

## 3. GitHub Secrets Reference

Configure secrets in your Flutter project under **Settings → Secrets and variables → Actions**:

| Secret | Target Workflow | Description |
|--------|-----------------|-------------|
| `KEYSTORE_PASSWORD` | Android | Password for Android keystore |
| `KEY_PASSWORD` | Android | Password for Android key |
| `KEY_ALIAS` | Android | Android signing key alias |
| `PLAY_STORE_SERVICE_ACCOUNT` | Play Store | Service Account JSON from Google Play Console |
| `APPSTORE_ACCOUNT_ID` | iOS | Branch name in certificates repository |
| `CERTIFICATES_REPO_TOKEN` | iOS | Personal Access Token with read access to certificates repo |
| `FIREBASE_SERVICE_ACCOUNT` | Android / iOS / PR | Optional Firebase Service Account JSON |
| `PAT_TOKEN` | All | Optional PAT for private Git dependencies |

---

## 4. Repository Structure

```
flutter-workflows/
├── README.md                                    # Comprehensive English documentation
├── iOS-README.md                                # Detailed iOS & Fastlane Match setup guide
├── SETUP.md                                     # Quick start & cheat sheet
├── .github/workflows/
│   ├── flutter-android-build.yml               # Android APK & AAB build
│   ├── flutter-ios-build.yml                   # iOS signed IPA build
│   ├── flutter-windows-build.yml               # Windows EXE & MSIX build
│   ├── flutter-pr-validation.yml               # PR fast validation
│   └── play-store-upload.yml                   # Google Play Store upload
├── example-combined-workflow.yml                # Example: Android Build + Upload
├── example-usage.yml                            # Example: Android Build only
├── example-pr-workflow.yml                     # Example: PR validation
└── example-play-store-upload.yml               # Example: Play Store upload only
```

For full details, inputs, and advanced customization, see [README.md](README.md).
