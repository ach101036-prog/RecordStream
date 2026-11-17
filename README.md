# TikLive Recorder

Android app that records TikTok live streams to local MP4 files for offline viewing.

## Features
- Enter a TikTok username and check whether the account is LIVE before recording
- Live HLS preview player (ExoPlayer) while recording
- Elapsed time counter + ongoing notification with a STOP action
- Remuxed MP4 saved locally (stream copy - no re-encoding, low CPU/battery)
- Notification alert when the file is saved, with the full path
- Saved to app storage: Android/data/com.example.tikliverecorder/files/Movies (copy via USB)

## How it works
1. TikTokLiveApi resolves username -> roomId from the public TikTok web page,
   then queries the live-detail endpoint (status 2 = live) for the HLS (m3u8) URL.
2. RecordService (foreground service) runs ffmpeg with -c copy to remux the HLS
   stream directly into an MP4 file.
3. A shared StateFlow drives the timer and save-status UI in MainActivity.

> WARNING: the TikTok endpoints are unofficial and undocumented - they can break
> at any time. The app shows errors gracefully instead of crashing.
>
> Only record streams you are authorized to record (your own, or with the
> creator's permission). Respect TikTok's Terms of Service and copyright law.

## Build locally
1. Open the project in Android Studio (Hedgehog or newer) and let Gradle sync.
2. Build > Build APK(s), or run: ./gradlew assembleDebug
3. APK output: app/build/outputs/apk/debug/app-debug.apk
   (debug builds are signed with the debug key and install directly)

If your checkout has no gradle wrapper binary, generate it once:
    gradle wrapper --gradle-version 8.7
(Android Studio does this automatically on first open.)

## Publish to GitHub
    git init
    git add .
    git commit -m "Initial commit"
    git branch -M main
    git remote add origin https://github.com/YOUR_USERNAME/TikLiveRecorder.git
    git push -u origin main

The included workflow (.github/workflows/build-apk.yml) builds an APK on every
push - download it from Actions > run > Artifacts.

## Release signing (optional, for wider distribution)
    keytool -genkey -v -keystore release.keystore -alias tiklive       -keyalg RSA -keysize 2048 -validity 10000

Then add a signingConfigs block in app/build.gradle.kts and store the keystore
as a GitHub encrypted secret if you build releases via CI.

## Stack
Kotlin - OkHttp - Media3/ExoPlayer (HLS preview) - ffmpeg-kit (MP4 remux) -
Foreground Service + Notifications - ViewBinding
