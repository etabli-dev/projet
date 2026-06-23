# Établi Projet — Android submission checklist

## Build outputs
- **Debug APK** — `app/build/outputs/apk/debug/app-debug.apk`
  - App ID `com.raban.etabli.projet.debug` (`.debug` suffix so it coexists with release).
- **Release AAB** — `app/build/outputs/bundle/release/app-release.aab`
  - R8 + resource shrinking. Signed with `app/upload-keystore.jks`.
  - **Upload this** to Play Console.

## Signing
Keystore at `app/upload-keystore.jks`; credentials in `keystore.properties` (gitignored).

> ⚠ The keystore was generated locally with a placeholder password (`android`). Before publishing, regenerate it with a strong password (or use Play App Signing). **Back up `upload-keystore.jks`** — losing it means losing the ability to ship updates to the same app.

```bash
keytool -genkeypair -v -keystore app/upload-keystore.jks -alias upload \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=Your Name, O=Your Org, C=DE"
```

## What's done (technical)
- Target SDK 34, min SDK 26
- Kotlin 1.9.24 + Compose BOM 2024.06.00 (Coder DS theme byte-identical with the rest of the suite)
- DataStore Preferences for persisted state
- OpenProject HAL+JSON client via OkHttp
- Bearer-token auth; token stored in DataStore (replace with EncryptedSharedPreferences for production hardening)
- Edge-to-edge insets via `enableEdgeToEdge()`
- Light + Dark themes (system-following)
- R8 + resource shrinking on release
- Proguard rules keep DTO field names so JSON wire format is stable across upgrades

## What you must do before publishing
- [ ] **Real upload key** — see signing section
- [ ] **Privacy policy URL** — required for Play Store
- [ ] **Store listing assets**:
  - Short description (≤80 chars) and full description (≤4000 chars)
  - 512×512 PNG hi-res icon
  - 1024×500 PNG feature graphic
  - ≥2 phone screenshots (1080×1920+ portrait)
- [ ] **Content rating questionnaire** — IARC, ~5 min in Play Console
- [ ] **Data safety form** — declare what's collected (user-entered server URL + API token. Personal/work data flows directly between this app and YOUR OpenProject instance. No data ever reaches us.)
- [ ] **Target audience and content** — pick "Adults" unless intended for children
- [ ] **App category** — "Productivity"
- [ ] **Pricing & distribution** — free/paid + countries
- [ ] **AD_ID exclusion** — confirm in Play Console that you don't use ads

## Test before upload
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.raban.etabli.projet.debug/com.raban.etabli.projet.MainActivity

# To install the release AAB locally:
# brew install bundletool
bundletool build-apks --bundle=app/build/outputs/bundle/release/app-release.aab \
  --output=/tmp/projet.apks --mode=universal
bundletool install-apks --apks=/tmp/projet.apks
```
