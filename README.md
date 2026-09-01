# google-sign-in SPM repro

Minimal reproduction for: **`@capawesome/capacitor-google-sign-in@0.1.3` npm tarball is
unparseable by SwiftPM** — the published `Package.swift` declares a test target at
`ios/PluginTests`, but the `files` array in `package.json` omits that directory from the
npm tarball.

Fails identically in Xcode (File → Packages → Resolve Package Dependencies does nothing)
and from the CLI.

## Environment

- macOS with Xcode / Command Line Tools (for `swift`)
- Node.js (any recent version)

## Repro (minimal — no Capacitor app needed)

```bash
npm install
cd node_modules/@capawesome/capacitor-google-sign-in
swift package describe
```

**Actual output:**

```
error: invalid custom path 'ios/PluginTests' for target 'GoogleSignInPluginTests'
```

**Expected output (if the tarball were consistent with its manifest):** the package
description (name, targets, dependencies) — no error.

## Repro (full app path)

```bash
npm install
npm install @capacitor/ios
npx cap add ios
npm run build
npx cap sync ios
# open ios/App/App.xcodeproj in Xcode
# File -> Packages -> Resolve Package Dependencies  →  no reaction (manifest parse failure)
```

## Root cause

`packages/google-sign-in/package.json` (in `capawesome-team/capacitor-plugins`):

```json
"files": [
  "android/src/main/",
  "android/build.gradle",
  "dist/",
  "ios/Plugin/",
  "CapawesomeCapacitorGoogleSignIn.podspec",
  "Package.swift"
]
```

`ios/PluginTests/` is **not** in `files`, so it is absent from the published tarball —
yet the published root `Package.swift` declares:

```swift
.testTarget(name: "GoogleSignInPluginTests", dependencies: [...], path: "ios/PluginTests")
```

The directory **does exist in the repository** (`packages/google-sign-in/ios/PluginTests/`),
so only the published artifact is inconsistent with its own manifest.

Suggested fix: add `"ios/PluginTests/"` to `files` (or drop the test target from the
published manifest).
