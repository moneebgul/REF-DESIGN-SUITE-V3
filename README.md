# RefDesign — Android app package

The full RefDesign cooling-system design tool, packaged two ways:

1. **As an installable PWA** — runs on the phone with no build step (fastest).
2. **As a Capacitor Android project** — produces a real, offline `.apk` you can install or distribute to the plants.

The tool itself is the single file `www/index.html` (works offline; no server calls).

---

## Path A — Use it now as a PWA (no Android Studio, no build)

The quickest way to get it on a phone home screen.

1. Put the `www/` folder somewhere it can be served over **https** (any of these):
   - GitHub Pages / Netlify / your intranet web server, **or**
   - on a PC: `cd www && python -m http.server 8080`, then open `http://<pc-ip>:8080` from the phone on the same network.
2. Open the page in **Chrome on Android**.
3. Menu (⋮) → **Add to Home screen** / **Install app**.
4. It installs with the RefDesign icon, opens full-screen, and works **offline** afterwards (service worker caches everything).

> A service worker needs https or localhost — opening the raw `index.html` as a `file://` won't enable install/offline. For a true offline file-based app with no hosting, use Path B.

---

## Path B — Build a real APK with Android Studio (recommended for distribution)

**Prerequisites:** Node.js 18+, Android Studio (includes the Android SDK + Gradle + JDK).

```bash
# 1. from this folder:
npm install                 # restores Capacitor (node_modules)
npx cap sync android        # copies www/ into the native project & links plugins

# 2. open the native project
npx cap open android        # launches Android Studio on the android/ folder
```

In Android Studio:
- Let Gradle finish syncing (first run downloads the Gradle distribution + SDK bits).
- **Build → Build Bundle(s)/APK(s) → Build APK(s)**.
- The debug APK lands at:
  `android/app/build/outputs/apk/debug/app-debug.apk`
- Install on a phone (enable *Install unknown apps*) or share that file.

App id: `com.dawlance.refdesign` · name: **RefDesign**

---

## Path C — Build from the command line

With the Android SDK installed and `ANDROID_HOME`/`ANDROID_SDK_ROOT` set:

```bash
npm install
npx cap sync android
cd android
./gradlew assembleDebug          # -> app/build/outputs/apk/debug/app-debug.apk
# release (needs a signing config in android/app/build.gradle):
# ./gradlew assembleRelease
```

### Signing a release APK (for sharing outside dev)
1. Create a keystore once:
   `keytool -genkey -v -keystore refdesign.keystore -alias refdesign -keyalg RSA -keysize 2048 -validity 10000`
2. Add a `signingConfigs`/`release` block in `android/app/build.gradle` pointing to it.
3. `./gradlew assembleRelease` → `app/build/outputs/apk/release/app-release.apk`.

---

## Path D — No Android Studio at all (cloud build)

1. Host the `www/` folder over https (GitHub Pages is free).
2. Go to **https://www.pwabuilder.com**, paste the URL.
3. It validates the PWA and generates a signed **Android package (.apk / .aab)** for you to download.

---

## Updating the tool later

The whole app is `www/index.html`. Replace it with a newer version, then:
```bash
npx cap sync android      # re-copies into the native project
```
and rebuild. Bump `CACHE` in `www/sw.js` (e.g. `refdesign-v6`) so installed PWAs pick up the new version.

---

## What's inside
```
www/                 the tool (index.html), manifest.json, sw.js, icons
android/             native Android (Capacitor) project — open in Android Studio
capacitor.config.json
package.json
```
`node_modules/` is intentionally not included — `npm install` restores it.
