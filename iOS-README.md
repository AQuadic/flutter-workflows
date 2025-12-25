# iOS CI/CD Setup Guide
## Flutter + Fastlane Match + GitHub Actions

---

## 📋 جدول المحتويات

1. [نظرة عامة](#نظرة-عامة)
2. [المتطلبات](#المتطلبات)
3. [الخطوة 1: إعداد Fastlane](#الخطوة-1-إعداد-fastlane)
4. [الخطوة 2: إعداد Fastlane Match](#الخطوة-2-إعداد-fastlane-match)
5. [الخطوة 3: App Store Connect API Key](#الخطوة-3-app-store-connect-api-key)
6. [الخطوة 4: إضافة تطبيق جديد لأكونت موجود](#الخطوة-4-إضافة-تطبيق-جديد-لأكونت-موجود)
7. [الخطوة 5: إضافة أكونت Apple جديد](#الخطوة-5-إضافة-أكونت-apple-جديد)
8. [الخطوة 6: GitHub Secrets](#الخطوة-6-github-secrets)
9. [الخطوة 7: استخدام الـ Workflow](#الخطوة-7-استخدام-الـ-workflow)
10. [مشاكل شائعة وحلولها](#مشاكل-شائعة-وحلولها)
11. [ملاحظات مهمة](#ملاحظات-مهمة)

---

## نظرة عامة

### الفكرة الكاملة

```
ios-certificates repo (private)
├── branch: client-one      ← Apple Account: Client 1
│   ├── certs/              ← Distribution Certificate (مشفر)
│   ├── profiles/           ← Provisioning Profiles (مشفر)
│   └── config.json         ← API Key + Match Password
├── branch: client-two      ← Apple Account: Client 2
│   ├── certs/
│   ├── profiles/
│   └── config.json
└── branch: client-xyz      ← أي أكونت جديد
    └── ...
```

كل تطبيق في GitHub بيحتاج **secret واحد بس**:
```
APPSTORE_ACCOUNT_ID = client-one   ← اسم الـ branch
```

وكل الباقي (certificates, API keys, passwords) بيتجيب تلقائياً من الـ `ios-certificates` repo.

---

## المتطلبات

- macOS مع Xcode مثبت
- Homebrew مثبت
- حساب Apple Developer Program (مدفوع)
- حساب GitHub مع وصول على الـ `ios-certificates` repo
- Flutter مثبت مع FVM

---

## الخطوة 1: إعداد Fastlane

### تثبيت Fastlane

```bash
brew install fastlane
```

تأكد من التثبيت:
```bash
fastlane --version
# fastlane 2.233.1
```

### تهيئة Fastlane في المشروع

```bash
cd /path/to/your/flutter/project/ios
fastlane init
```

> اختر **4 (Manual setup)** لما يسألك

هيتعمل:
```
ios/
├── Gemfile
├── Gemfile.lock
└── fastlane/
    ├── Appfile
    └── Fastfile
```

> ⚠️ **مهم:** اعمل commit لهذه الملفات في الـ repo

```bash
git add ios/Gemfile ios/Gemfile.lock ios/fastlane/
git commit -m "chore: add fastlane configuration"
```

---

## الخطوة 2: إعداد Fastlane Match

### تهيئة Match

```bash
cd ios
fastlane match init
```

- اختر **1 (git)**
- أدخل URL الـ repo: `https://github.com/<your-org>/ios-certificates.git`

هيتعمل ملف `ios/fastlane/Matchfile`، عدّله ليبقى:

```ruby
git_url("https://github.com/<your-org>/ios-certificates.git")
storage_mode("git")
type("appstore")

# هيتحدد من الـ GitHub Actions تلقائياً
# git_branch("account-name")
# app_identifier("com.company.app")
```

> اعمل commit للـ Matchfile:
```bash
git add ios/fastlane/Matchfile
git commit -m "chore: add match configuration"
```

### إنشاء Certificates للتطبيق

```bash
fastlane match appstore \
  --app_identifier "com.your.bundleid" \
  --username "apple-account@email.com" \
  --git_branch "account-branch-name"
```

**أثناء التشغيل:**
1. هيطلب **Passphrase** — اخترله password قوي واحتفظ بيه ⚠️ (ده الـ `match_password`)
2. هيطلب **Apple ID Password**
3. هيطلب **2FA Code** على تليفونك

> ⚠️ **مهم جداً:** بعد الانتهاء، روح على GitHub → `ios-certificates` → **غيّر اسم الـ branch** من `master` للاسم الصح (مثلاً `my-app`)
>
> **Settings → Branches → Rename**

---

## الخطوة 3: App Store Connect API Key

### إنشاء الـ API Key

1. روح على [App Store Connect](https://appstoreconnect.apple.com)
2. **Users and Access → Integrations → App Store Connect API**
3. اضغط **+** لإنشاء key جديد
4. اديه اسم وصلاحية **Developer**

احتفظ بـ:
- ✅ **Key ID** (مثلاً: `ABC123DEFG`)
- ✅ **Issuer ID** (مثلاً: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
- ✅ **ملف `.p8`** — ⚠️ **بينزل مرة واحدة بس، احتفظ بيه جيداً**

### إنشاء config.json

روح على GitHub → `ios-certificates` → غيّر الـ branch للأكونت المطلوب → افتح ملف جديد باسم `config.json`:

```json
{
  "issuer_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "key_id": "ABC123DEFG",
  "key_content": "-----BEGIN PRIVATE KEY-----\nMIGTAgEAMBMG...\n-----END PRIVATE KEY-----",
  "match_password": "your-strong-match-password"
}
```

> **ملاحظة الـ key_content:** افتح ملف الـ `.p8` بأي text editor، واستبدل كل سطر جديد بـ `\n`

---

## الخطوة 4: إضافة تطبيق جديد لأكونت موجود

لو عندك تطبيق جديد على **نفس الأكونت** (مثلاً أكونت my-app):

```bash
cd /path/to/new/flutter/project/ios
fastlane init   # اختر 4
fastlane match init   # نفس الـ repo URL

fastlane match appstore \
  --app_identifier "com.company.newapp" \
  --username "apple-account@email.com" \
  --git_branch "my-app"
```

هيضيف الـ provisioning profile الجديد في نفس الـ branch تلقائياً.

**في repo التطبيق:**
```
APPSTORE_ACCOUNT_ID = my-app          ← نفس الـ branch
CERTIFICATES_REPO_TOKEN = ghp_xxx     ← نفس الـ PAT
```

---

## الخطوة 5: إضافة أكونت Apple جديد

لو عندك **أكونت Apple جديد كلياً**:

```bash
fastlane match appstore \
  --app_identifier "com.newclient.app" \
  --username "newclient@email.com" \
  --git_branch "newclient"
```

بعدين:
1. غيّر الـ branch من `master` لـ `newclient` على GitHub
2. اعمل `config.json` جديد في الـ branch ده بـ API Key بتاع الأكونت الجديد
3. في repo التطبيق:
```
APPSTORE_ACCOUNT_ID = newclient
```

---

## الخطوة 6: GitHub Secrets

### Personal Access Token (PAT)

محتاج PAT عشان الـ GitHub Actions runner يقدر يقرأ الـ `ios-certificates` repo.

**إنشاء PAT:**
1. GitHub → Settings → Developer Settings → Personal Access Tokens → **Fine-grained tokens**
2. اضغط **Generate new token**
3. الإعدادات:
   ```
   Resource owner:     <your-org>
   Repository access:  Only selected → ios-certificates
   Permissions:
     Contents: Read-only
   ```
4. احتفظ بالـ token

### Secrets في كل repo تطبيق

**Settings → Secrets and variables → Actions → New repository secret**

| Secret | القيمة | ملاحظة |
|--------|--------|--------|
| `APPSTORE_ACCOUNT_ID` | `my-app` | اسم الـ branch في ios-certificates |
| `CERTIFICATES_REPO_TOKEN` | `ghp_xxxx` | الـ PAT اللي عملته |
| `FIREBASE_SERVICE_ACCOUNT` | `{...json...}` | **مطلوب** لو بتستخدم `firebase-project-id` |

> ⚠️ **Firebase:** لو بتمرر `firebase-project-id` كـ input في الـ workflow، الـ `FIREBASE_SERVICE_ACCOUNT` secret بيبقى **إجباري**. لو مش موجود، الـ step هيتخطى مع warning.

---

## الخطوة 7: استخدام الـ Workflow

### في repo التطبيق، اعمل ملف `.github/workflows/ios.yml`:

```yaml
name: iOS Build

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:

jobs:
  build:
    uses: aquadic/flutter-workflows/.github/workflows/flutter-ios-build.yml@main
    with:
      # اختياري - repo الشهادات (افتراضياً: <your-org>/ios-certificates)
      # certificates-repo: "my-org/ios-certificates"

      # اختياري - لو مش محدد بيتعمل auto-detect من Xcode
      # bundle-id: "com.company.myapp"
      
      # اختياري
      # app-name: "My App"
      
      # اختياري - لو عندك Firebase
      # firebase-project-id: "my-firebase-project"
      
      # اختياري - لو عايز runner معين
      # runner-type: "macos-latest"
      
    secrets:
      APPSTORE_ACCOUNT_ID: ${{ secrets.APPSTORE_ACCOUNT_ID }}
      CERTIFICATES_REPO_TOKEN: ${{ secrets.CERTIFICATES_REPO_TOKEN }}
      
      # مطلوب لو بتستخدم firebase-project-id
      # FIREBASE_SERVICE_ACCOUNT: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

---

## مشاكل شائعة وحلولها

### ❌ Bundle ID اتكشف كـ `NO`

**السبب:** `xcodebuild -showBuildSettings` بيرجع `DERIVE_MACCATALYST_PRODUCT_BUNDLE_IDENTIFIER = NO` وبياخد الـ `NO` كـ bundle ID.

**الحل:** الـ workflow بيستخدم `-workspace Runner.xcworkspace` مع `grep -v 'DERIVE_MACCATALYST'` عشان يتجنب ده.

لو لسه بيحصل، حدد الـ `bundle-id` يدوياً كـ input.

---

### ❌ `Error: Invalid format` في الـ key_content

**السبب:** الـ `key_content` فيه newlines والـ GitHub Output مش بيتعامل معاهم صح لو اتمرروا كـ step output.

**الحل:** الـ workflow بيكتب الـ key في ملف مباشرة:
```bash
echo "$CONFIG" | jq -r '.key_content' > /tmp/api_key.p8
```
وبيستخدم `"key_filepath"` بدل `"key"` في الـ `api_key.json`.

---

### ❌ `repository not found` في Match

**السبب:** الـ `github.token` مش عنده access على الـ `ios-certificates` repo.

**الحل:** استخدم `CERTIFICATES_REPO_TOKEN` (الـ PAT) في الـ git_url:
```bash
--git_url "https://x-access-token:${CERTIFICATES_REPO_TOKEN}@github.com/<your-org>/ios-certificates.git"
```

---

### ❌ Match بيحفظ على branch `master` بدل الاسم المطلوب

**السبب:** الـ `--git_branch` flag بيتجاهل أحياناً في أول run.

**الحل:** بعد أول `fastlane match appstore`، روح على GitHub وغيّر اسم الـ `master` branch يدوياً للاسم الصح من **Settings → Branches → Rename**.

---

### ❌ `Profile not found` أثناء الـ export

**السبب:** اسم الـ profile في `ExportOptions.plist` مش مطابق للي عنده Match.

**الحل:** Match دايماً بيسمي الـ profiles بالصيغة دي:
```
match AppStore com.your.bundleid
```
الـ workflow بيبنيه تلقائياً من الـ bundle ID.

---

### ❌ `pod install` بيفشل على الـ runner

**السبب:** الـ Pods مش متكاشة أو الـ `Podfile.lock` اتغير.

**الحل:** الـ workflow عنده cache للـ CocoaPods مرتبط بـ `Podfile.lock`. لو الـ Podfile.lock اتغير، الـ cache بينكسر وبيعمل `pod install` جديد تلقائياً.

---

### ❌ خطأ في الـ `sed` command

**السبب:** macOS بتستخدم BSD `sed` اللي بتختلف عن Linux `sed`.

**الحل:** الـ workflow بيستخدم `sed -i ''` (مع quotes فارغة) اللي هو الصيغة الصح لـ macOS.

---

## ملاحظات مهمة

### 🔐 الأمان

- الـ `ios-certificates` repo فيها ملفات **مشفرة** بالـ `match_password` — حد يشوف الـ repo من غير الـ password مش هيقدر يعمل حاجة
- الـ `config.json` فيه الـ `match_password` و API Key — الـ repo لازم تكون **private** دايماً
- الـ PAT عنده **read-only** access على `ios-certificates` بس

### 🔄 تجديد الـ Certificates

الـ Distribution Certificate صالح لـ **سنة**. لما بينتهي:

```bash
fastlane match appstore \
  --app_identifier "com.your.bundleid" \
  --username "apple@email.com" \
  --git_branch "account-branch" \
  --force
```

### 📱 إضافة Bundle ID جديد لأكونت موجود

مش محتاج تعمل حاجة في الـ config، بس شغّل:

```bash
fastlane match appstore \
  --app_identifier "com.company.newapp" \
  --username "apple@email.com" \
  --git_branch "my-app"
```

### ⚡ أول build دايماً أبطأ

الـ cache (Flutter + CocoaPods) بيتبني في أول run. من الـ run التاني، الـ build هيبقى أسرع بكتير.

### 🔑 لو نسيت الـ match_password

مفيش طريقة تسترجعه. هتحتاج:
1. تمسح كل الـ certificates من الـ Apple Developer Portal
2. تمسح محتوى الـ branch في `ios-certificates`
3. تشغّل `fastlane match appstore` من الأول

**احتفظ بالـ password في مكان آمن (مثلاً 1Password أو LastPass).**
