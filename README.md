# Flutter Workflows

Reusable GitHub Actions workflows for Flutter projects.

## Available Workflows

1. [🤖 Flutter Android Build](#-flutter-android-build) - Builds APK and AAB (App Bundle) with signing and obfuscation.
2. [🍏 Flutter iOS Build](#-flutter-ios-build) - Builds signed IPA with Fastlane Match and App Store Connect integration.
3. [📤 Upload to Google Play](#-upload-to-google-play) - Automatically uploads AAB releases to Google Play Store tracks.
4. [🪟 Flutter Windows Build](#-flutter-windows-build) - Builds Windows executables, MSIX packages, and portable ZIP archives.
5. [🔍 Flutter PR Validation](#-flutter-pr-validation) - Fast CI checks (analyze, format, tests) for pull requests on self-hosted runners.

---

## 🤖 Flutter Android Build

Builds APK and AAB (App Bundle) for Android with optional obfuscation and signing.

**Location:** `.github/workflows/flutter-android-build.yml`

### Features:
- ✅ Builds both APK and AAB
- ✅ Optional ProGuard obfuscation & mapping upload
- ✅ Automated symbol uploads (Crashlytics)
- ✅ Automatic `build_runner` detection & execution
- ✅ Dynamic app renaming, bundle ID change, and icon download
- ✅ Configurable find-and-replace text replacement
- ✅ Firebase configuration support
- ✅ FVM & configurable Java version
- ✅ GitHub Release integration

### Usage:

**Simplest setup (recommended):**
```yaml
name: Build Android
on:
  push:
    tags:
      - "*.*.*"

jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-android-build.yml@main
    secrets: inherit  # ✅ All secrets automatically passed
```

**With custom options:**
```yaml
jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-android-build.yml@main
    secrets: inherit
    with:
      java-version: '17'              # optional, default: '17'
      enable-obfuscation: true        # optional, default: true
      runner-type: 'ubuntu-latest'    # optional, default: 'self-hosted'
      run-tests: true                 # optional, default: true
```

### Inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `java-version` | Java version to use | No | `'17'` |
| `enable-obfuscation` | Enable obfuscation for AAB | No | `true` |
| `runner-type` | Runner type (`self-hosted` / `ubuntu-latest`) | No | `'self-hosted'` |
| `run-tests` | Run unit tests before building if tests exist | No | `true` |
| `app-name` | Override app display name | No | `''` |
| `bundle-id` | Override Android package name | No | `''` |
| `icon-url` | Download custom app icon (PNG) | No | `''` |
| `find-replace-list` | Line-separated string of `path\|find\|replace` | No | `''` |
| `firebase-project-id` | Firebase project ID for configuration | No | `''` |
| `pre-build-script` | Custom script to run before build | No | `''` |

### Secrets (Optional):

| Secret | Description | Required |
|--------|-------------|----------|
| `KEYSTORE_PASSWORD` | Keystore password for signing | No |
| `KEY_PASSWORD` | Key password for signing | No |
| `KEY_ALIAS` | Key alias for signing | No |
| `FIREBASE_SERVICE_ACCOUNT` | Firebase Service Account JSON (required if `firebase-project-id` is used) | No |
| `PAT_TOKEN` | Personal Access Token for private Git dependencies | No |

---

## 🍏 Flutter iOS Build

Builds and signs iOS release packages (`.ipa`) using Fastlane Match certificates management and App Store Connect API.

**Location:** `.github/workflows/flutter-ios-build.yml`

### Features:
- ✅ Automated code signing via **Fastlane Match** (App Store profile)
- ✅ Centralized certificate management across multiple Apple accounts / teams
- ✅ Automatic Bundle ID detection & overrides
- ✅ CocoaPods & Swift Package Manager (SPM) dependency resolution
- ✅ Automatic App Store Connect API Key integration
- ✅ Optional Firebase setup and FlutterFire configuration
- ✅ Dynamic app renaming, custom icon download, and find-and-replace
- ✅ GitHub Release integration with `.ipa` upload

### Usage:

```yaml
name: Build iOS
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
      # certificates-repo: 'your-org/ios-certificates'  # optional: defaults to <owner>/ios-certificates
```

### Inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `certificates-repo` | GitHub repository containing iOS certificates (`owner/repo`) | No | `<owner>/ios-certificates` |
| `runner-type` | Runner type | No | `'macos-latest'` |
| `bundle-id` | Override iOS Bundle Identifier | No | `''` (auto-detected) |
| `app-name` | Override app display name | No | `''` |
| `icon-url` | Download custom app icon (PNG) | No | `''` |
| `find-replace-list` | Line-separated string of `path\|find\|replace` | No | `''` |
| `firebase-project-id` | Firebase project ID for configuration | No | `''` |
| `pre-build-script` | Script before build | No | `''` |
| `run-tests` | Run unit tests before building | No | `true` |

### Secrets:

| Secret | Description | Required |
|--------|-------------|----------|
| `APPSTORE_ACCOUNT_ID` | Branch name in certificates repo corresponding to the Apple account | **Yes** |
| `CERTIFICATES_REPO_TOKEN` | Personal Access Token with read access to the certificates repository | **Yes** |
| `FIREBASE_SERVICE_ACCOUNT` | Firebase Service Account JSON (required if `firebase-project-id` is used) | No |
| `PAT_TOKEN` | Personal Access Token for private Git dependencies | No |

> 📖 **Full iOS Setup Guide:** For an end-to-end walkthrough on setting up Fastlane Match, Apple API keys, and certificate repositories, check out [iOS-README.md](iOS-README.md) (in Arabic).

---

## 📤 Upload to Google Play

Automatically discovers `.aab` artifacts from GitHub Releases and uploads them to Google Play Store tracks.

**Location:** `.github/workflows/play-store-upload.yml`

### Features:
- ✅ Matrix upload for multiple AABs in a single release
- ✅ Uploads to any Google Play track (`internal`, `alpha`, `beta`, `production`)
- ✅ Uploads ProGuard `mapping.txt` and native debug symbols archive (`symbols.zip`)
- ✅ Automatically incorporates localized release notes from `whatsnew/` folder
- ✅ Automatically detects package name from AAB asset names

### Usage:

```yaml
name: Upload to Play Store
on:
  release:
    types: [published]

jobs:
  upload:
    uses: aquadic/flutter-workflows/.github/workflows/play-store-upload.yml@main
    secrets: inherit
    with:
      play_store_track: 'internal'
```

### Inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `play_store_track` | Target track (`internal`, `alpha`, `beta`, `production`) | No | `'internal'` |
| `use_latest` | Fetch latest release tag if not triggered directly by a tag | No | `false` |
| `release_tag` | Explicit release tag override | No | `''` |
| `runner-type` | Runner type | No | `'self-hosted'` |

### Secrets:

| Secret | Description | Required |
|--------|-------------|----------|
| `PLAY_STORE_SERVICE_ACCOUNT` | Google Play Console Service Account JSON (workflow auto-skips if missing) | **Yes** |

---

## 🪟 Flutter Windows Build

Builds Release Windows executables, official MSIX installers, and portable ZIP archives.

**Location:** `.github/workflows/flutter-windows-build.yml`

### Features:
- ✅ Builds Windows release binaries (`.exe`)
- ✅ Generates Windows MSIX packages (`.msix`) via `msix:create`
- ✅ Produces standalone portable archive (`windows-release.zip`)
- ✅ Automatic `build_runner` code generation
- ✅ FVM & Flutter channel configuration
- ✅ Automatic Release assets upload when triggered by a tag

### Usage:

```yaml
name: Build Windows
on:
  push:
    tags:
      - "*.*.*"

jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-windows-build.yml@main
    secrets: inherit
```

### Inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `enable-msix` | Generate MSIX package | No | `true` |
| `run-tests` | Run unit tests before building | No | `true` |
| `runner-type` | Runner type | No | `'windows-latest'` |
| `flutter-version` | Flutter version to use (overrides FVM) | No | `''` |
| `flutter-channel` | Flutter channel | No | `'stable'` |
| `app-name` | App name override | No | `''` |
| `find-replace-list` | Line-separated `path\|find\|replace` | No | `''` |
| `pre-build-script` | PowerShell script before build | No | `''` |

### Secrets (Optional):

| Secret | Description | Required |
|--------|-------------|----------|
| `PAT_TOKEN` | Personal Access Token for private Git dependencies | No |

---

## 🔍 Flutter PR Validation

Fast CI validation workflow designed for Pull Requests running on self-hosted or cloud runners. Executes all pre-build steps, code generators, analysis, and unit tests, skipping heavy release asset packaging.

**Location:** `.github/workflows/flutter-pr-validation.yml`

### Features:
- ✅ Runs on `self-hosted` runners by default (or cloud runners)
- ✅ Pre-build checks without compiling full binaries
- ✅ Automatic `build_runner` detection & execution
- ✅ FVM & Java configuration support
- ✅ Static analysis via `flutter analyze`
- ✅ Optional formatting checks (`dart format`)
- ✅ Automated unit testing with commit skip tags (`[skip tests]`, `[no ci]`)
- ✅ Safe git cache cleanup and credentials handling for self-hosted environments

### Usage:

```yaml
name: Pull Request Validation
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

### Inputs:

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `runner-type` | Runner type to use | No | `'self-hosted'` |
| `java-version` | Java version to use | No | `'17'` |
| `flutter-version` | Flutter version override (overrides FVM) | No | `''` |
| `flutter-channel` | Flutter channel | No | `'stable'` |
| `run-analyze` | Run `flutter analyze` | No | `true` |
| `run-tests` | Run unit tests | No | `true` |
| `run-format-check` | Check Dart code formatting | No | `false` |
| `app-name` | App Name for renaming | No | `''` |
| `bundle-id` | Package name / bundle id | No | `''` |
| `icon-url` | URL to download app icon | No | `''` |
| `find-replace-list` | Find & replace operations (`path\|find\|replace`) | No | `''` |
| `firebase-project-id` | Firebase project ID | No | `''` |
| `pre-build-script` | Script before validation | No | `''` |

### Secrets (Optional):

| Secret | Description | Required |
|--------|-------------|----------|
| `PAT_TOKEN` | Personal Access Token for private Git dependencies | No |
| `FIREBASE_SERVICE_ACCOUNT` | Firebase Service Account JSON | No |

---

## Example: Complete Multi-Platform Release

```yaml
# .github/workflows/release.yml
name: Full Release
on:
  push:
    tags:
      - "*.*.*"

jobs:
  build-android:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-android-build.yml@main
    secrets: inherit

  build-ios:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-ios-build.yml@main
    secrets: inherit

  upload-play-store:
    needs: build-android
    uses: aquadic/flutter-workflows/.github/workflows/play-store-upload.yml@main
    secrets: inherit
```

---

## Contributing

1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/my-feature`).
3. Commit your changes (`git commit -m 'feat: add new feature'`).
4. Push to the branch (`git push origin feature/my-feature`).
5. Open a Pull Request.

## License

MIT
