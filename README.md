# react-native-splash-motion iOS repro

A blank Expo SDK 58 app (`create-expo-app --template blank-typescript`) with
`react-native-splash-motion` (0.1.1 on `main`, 0.1.2 on the `app-extension` branch), `react-native-nitro-modules@0.37.1` and
`expo-dev-client`. The animation is the library's own `example/assets/splash.json`.
`App.tsx` calls `useHideSplash()`.

## 1. The launch storyboard fails to compile when `still` is set

`app.json` sets `still: ./assets/splash-first-frame.png`.

```sh
npm install
npx expo prebuild -p ios
npx expo run:ios
```

The build fails in `CompileStoryboard`:

```
ios/splashmotionrepro/SplashScreen.storyboard: error: Line 29: Opening and ending tag mismatch: view line 13 and constraints
```

`xmllint --noout ios/splashmotionrepro/SplashScreen.storyboard` reports the same error.

## 2. The overlay covers the expo-dev-client launcher

Remove the `still` option from `app.json` (to get past issue 1), then build a
Debug development build and open it **without** a running dev server, so the
launcher shows its own screen:

```sh
npx expo prebuild -p ios --clean
npx expo run:ios --no-bundler
```

The splash plays on top of the launcher ("Development Build", "Searching for
development servers…") and never goes away. Taps on "Enter URL manually" or on a
server do nothing. See [media/dev-launcher-covered.mp4](media/dev-launcher-covered.mp4).

## 3. An app extension's folder is used instead of the app's (`app-extension` branch)

This branch uses 0.1.2 and adds `expo-widgets` with one widget, which creates an
app extension in `ios/ExpoWidgetsTarget`. The generator picks the first folder in
`ios/` with an `Info.plist`, and that is the widget's:

```sh
git checkout app-extension
npm install
npx expo prebuild -p ios --clean
# Error: [ios.dangerous]: withIosDangerousBaseMod: [splash-motion] No .xcassets catalog found in ./ios/ExpoWidgetsTarget
```

If the extension has an asset catalog, there is no error, and the splash is written
into the extension instead:

```sh
mkdir ios/ExpoWidgetsTarget/Assets.xcassets
npx splash-motion generate assets/splash.json --background "#0F6E56" --width 160
#   ✓ iOS      assets + ios/ExpoWidgetsTarget/LaunchScreen.storyboard
```

With the fix (`findIosApp()` skipping app extensions), the prebuild passes, the
splash lands in `ios/splashmotionrepro`, and a Release build plays it:
[media/app-extension-fixed.mp4](media/app-extension-fixed.mp4).
