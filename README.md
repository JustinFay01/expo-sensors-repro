# expo-sensors TCC Kill Repro (iOS 26)

Minimal reproduction for: **`[expo-sensors] App killed by TCC on iOS 26 when motionPermission: false`**

## The Bug

On iOS 26, setting `motionPermission: false` in the expo-sensors config plugin causes the app to be killed by TCC (Transparency, Consent, and Control) when `Accelerometer.addListener` is called.

Previously, `CMMotionManager` accelerometer APIs did not require Motion & Fitness permission. iOS 26 has extended TCC enforcement to cover all CoreMotion access, including raw accelerometer data.

### What happens

1. `motionPermission: false` correctly excludes `EXMotionPermissionRequester.m` and omits `NSMotionUsageDescription` from Info.plist
2. `Accelerometer.addListener` calls `CMMotionManager.startAccelerometerUpdates()`
3. iOS 26 TCC gate triggers for `kTCCServiceMotion`
4. No `NSMotionUsageDescription` in Info.plist → TCC refuses → process killed (exit domain: `tcc(11)`)

### Key log lines

```
tccd  AUTHREQ_PROMPTING: service=kTCCServiceMotion
tccd  Refusing authorization request for service kTCCServiceMotion ... without NSMotionUsageDescription key
SpringBoard  Process exited: domain:tcc(11) code:0
```

## Environment

- iOS 26.3.1 (a) on iPad Pro M4
- Expo SDK 55 (`~55.0.20`)
- expo-sensors `~55.0.13`
- React Native `0.83.6`

## Reproduce

```bash
npm install
npx expo prebuild --platform ios --clean
npx expo run:ios --device
```

Once running, press `r` in Metro to reload. The app will be killed by TCC.

## Config verification

After prebuild, confirm the plugin applied correctly:

- `ios/Podfile.properties.json` contains `"MOTION_PERMISSION": "false"`
- `ios/exposensorsrepro/Info.plist` does NOT contain `NSMotionUsageDescription`
- `EXMotionPermissionRequester.m` is excluded from compilation
