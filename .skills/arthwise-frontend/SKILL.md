---
name: arthwise-frontend
description: Core context and architecture for the Arthwise frontend mobile app (clientapp). Trigger when the user asks about the frontend, mobile app, React Native, Expo, UI components, or Android/iOS builds.
---

# Arthwise Frontend Knowledge

## Architecture Overview
The frontend is a React Native mobile application built with Expo, located in the `clientapp` directory.

## Key Technologies
- **Framework**: React Native with Expo (SDK 54+).
- **Navigation**: React Navigation (Stack, Bottom Tabs, Top Tabs, Drawer).
- **State Management**: Redux Toolkit (in `client/store/`) combined with React Query (`@tanstack/react-query`) for API data fetching and caching.
- **UI Components**: React Native Paper.
- **Charting**: Lightweight Charts (`lightweight-charts`) and Victory Native.
- **Animations**: Lottie (`lottie-react-native`).
- **Backend Communication**: Axios and Socket.io-client.

## Services & Integrations
- **Firebase**: Heavily integrated for Analytics, Crashlytics, In-App Messaging, and Cloud Messaging (FCM). 
  - Configuration files: `firebaseConfig.js`, `google-services.json` (Android), and `GoogleService-Info.plist` (iOS).
- **Authentication**: Uses Google Sign-In (`@react-native-google-signin/google-signin`).
- **Deep Linking**: Configured in `app.json` with scheme `arthwise` and app links to `arthhwise.com`.

## Common Workflows
- **Running Locally**:
  - Start Expo dev server: `npm start` or `npx expo start`.
  - Run on Android emulator: `npm run android`.
  - Run on iOS simulator: `npm run ios`.
- **Building & Deployment**:
  - Handled via cd android && ./gradlew bundleRelease creats .aab file and then pushed to play console
  - Configurations in `eas.json` and `app.json`.
  - Prebuild is used (`npx expo prebuild`) for native module linking when needed.
- **Debugging**:
  - Global error handling setup in `App.js` with custom Error Boundaries.
  - Network and Socket connection managers monitor online status.

## Directory Structure Highlights
- `App.js`: Root entry point, sets up Providers (Redux, QueryClient), Error Boundaries, and Notifications.
- `client/`: Main source code directory (screens, components, hooks, utils).
- `client/store/`: Redux configuration and slices.
- `assets/`: Images, icons, and splash screens.
