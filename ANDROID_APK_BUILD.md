# Build the SignBridge Android APK

## Hosted build (recommended)

1. Create a GitHub repository and push the entire project folder contents to `main`.
2. Enable Actions if GitHub asks.
3. Go to Actions → **Build SignBridge Android APK** → Run workflow.
4. Wait for a successful run, then download the `SignBridge-Android-debug-APK` artifact.
5. Extract the artifact ZIP; it contains `app-debug.apk`.
6. Transfer the APK to Android and install it, granting camera access when prompted.

The GitHub Actions workflow prepares Capacitor Android, installs Java 21 and Android SDK 36, then compiles a debug APK. It runs on pushes that touch the app/build files and on manual workflow dispatch.

## Local build

Requirements: Node.js 22+, JDK 21, Android SDK Platform 36 and Build Tools 36.0.0.

```bash
./scripts/build-android.sh
```

The output is `android/app/build/outputs/apk/debug/app-debug.apk`.

## Android version

The provided stable build targets API 36 and is designed to run on Android 17 devices. It is not a target-API-37 build. Use a matching newer Android Studio/Android Gradle Plugin and test the API 37 behavior changes before changing compile/target SDK. Debug APKs are for testing; create a signed release APK/AAB for distribution. Do not share signing keys.

## Why no APK binary is attached yet

This runtime has Java 21 but no Android SDK, no Gradle executable or generated Android platform, and cannot resolve/download the Android SDK command-line tools. The APK cannot be honestly reported as compiled here; use the supplied GitHub Actions build to produce the real binary.
