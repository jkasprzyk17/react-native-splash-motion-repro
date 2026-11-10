# react-native-splash-motion iOS repro

A blank Expo SDK 58 app (`create-expo-app --template blank-typescript`) with
`react-native-splash-motion@0.1.1`, `react-native-nitro-modules@0.37.1` and
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
