/**
 * SETUP CHECKLIST: Firebase Cloud Messaging (FCM)
 * Complete step-by-step setup guide
 */

// ============================================================
// STEP 1: GET FIREBASE SERVICE ACCOUNT KEY
// ============================================================

/**
 * You need the Firebase Admin SDK service account key to send notifications
 * from your backend.
 * 
 * 1. Go to Firebase Console: https://console.firebase.google.com
 * 2. Select your Arthwise project
 * 3. Go to Project Settings (⚙️ icon) > Service Accounts
 * 4. Click "Generate new private key"
 * 5. Save the JSON file securely
 * 
 * This file looks like:
 * {
 *   "type": "service_account",
 *   "project_id": "arthwise-xxxxx",
 *   "private_key_id": "...",
 *   "private_key": "-----BEGIN PRIVATE KEY-----\\n...",
 *   "client_email": "firebase-adminsdk-xxxxx@arthwise-xxxxx.iam.gserviceaccount.com",
 *   ...
 * }
 * 
 * ⚠️  SECURITY: Never commit this file to Git!
 * Add to .gitignore: serviceAccountKey.json
 */

// ============================================================
// STEP 2: INSTALL BACKEND DEPENDENCIES
// ============================================================

/**
 * In your ArthwiseServices directory:
 * 
 * npm install firebase-admin
 * npm install dotenv (for environment variables)
 */

// ============================================================
// STEP 3: CONFIGURE ENVIRONMENT VARIABLES
// ============================================================

/**
 * Create .env file in ArthwiseServices:
 * 
 * FIREBASE_PROJECT_ID=arthwise-xxxxx
 * FIREBASE_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----\n...
 * FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxxxx@arthwise-xxxxx.iam.gserviceaccount.com
 * 
 * Or load the entire service account JSON:
 * FIREBASE_SERVICE_ACCOUNT_KEY=/path/to/serviceAccountKey.json
 */

// ============================================================
// STEP 4: VERIFY APP CONFIGURATION FILES
// ============================================================

/**
 * Check these files in clientapp/:
 * 
 * ✅ package.json - Has @react-native-firebase/messaging
 * ✅ app.json - Has firebase plugins configured
 * ✅ android/ - Has google-services.json
 * ✅ ios/ - Has GoogleService-Info.plist
 * ✅ firebaseConfig.js - Updated with messaging
 */

// ============================================================
// STEP 5: UPDATE app.json FOR NOTIFICATIONS
// ============================================================

/**
 * Make sure your app.json has the notification plugin:
 * 
 * "plugins": [
 *   [
 *     "expo-notifications",
 *     {
 *       "icon": "./assets/notification-icon.png",
 *       "color": "#FF6B00",
 *       "sounds": [
 *         "./assets/notification-sound.wav"
 *       ],
 *       "defaultChannel": "default"
 *     }
 *   ],
 *   ...
 * ]
 */

// ============================================================
// STEP 6: REBUILD NATIVE MODULES
// ============================================================

/**
 * After installing firebase-admin and updating packages:
 * 
 * cd clientapp
 * npx expo prebuild --clean
 * 
 * This regenerates Android and iOS native code with new dependencies.
 */

// ============================================================
// STEP 7: TEST ON DEVICE
// ============================================================

/**
 * 1. Install app on Android device or iOS simulator
 * 2. Open app and check console for:
 *    ✅ "FCM initialized with token: ABC123..."
 * 3. Note down the FCM token
 * 4. Go to Firebase Console > Cloud Messaging
 * 5. Create new campaign
 * 6. Send test notification to this token
 * 7. Verify:
 *    - Notification appears when app is in foreground
 *    - Notification appears in tray when app is in background
 *    - Tapping notification opens app
 */

// ============================================================
// STEP 8: PRODUCTION SETUP - SENDING UPDATES
// ============================================================

/**
 * WORKFLOW for notifying users about app update:
 * 
 * 1. DEVELOP:
 *    - Create new features
 *    - Test in dev build
 *    - Increment version in app.json (e.g., 1.3.5 -> 1.4.0)
 * 
 * 2. BUILD & RELEASE:
 *    - Build release APK: eas build --platform android --release
 *    - Build release AAB: eas build --platform android --release-type release
 *    - Upload to Play Store Internal Testing
 *    - Wait 24 hours for Play Store processing
 *    - Promote to Production
 * 
 * 3. NOTIFY USERS:
 *    - Wait 1-2 hours for Play Store indexing
 *    - Run backend script: await sendAppUpdateNotification('1.4.0')
 *    - Or create campaign in Firebase Console
 * 
 * 4. MONITOR:
 *    - Check Firebase Console for delivery metrics
 *    - Monitor crash logs for compatibility issues
 *    - Watch user ratings for feedback
 */

// ============================================================
// STEP 9: IN-APP UPDATE GUIDANCE (OPTIONAL)
// ============================================================

/**
 * For forced updates, you can also embed update logic:
 * 
 * On app start, check backend for minimum required version:
 * 
 * GET /api/app/version
 * Response: {
 *   "latestVersion": "1.4.0",
 *   "minimumVersion": "1.3.0",
 *   "requiredVersion": null, // null = no force, "1.4.0" = force
 *   "updateUrl": "https://play.google.com/store/..."
 * }
 * 
 * If current version < requiredVersion -> Show full-screen update prompt
 * If current version < minimumVersion -> Show warning
 */

// ============================================================
// STEP 10: TROUBLESHOOTING
// ============================================================

/**
 * Issue: FCM token not appearing in console
 * Solution: Check app permissions in Android/iOS settings
 * 
 * Issue: Notification not received in background
 * Solution: Check notification channel in Android settings
 * 
 * Issue: Tokens keep changing
 * Solution: This is normal, always update backend with new token
 * 
 * Issue: "messaging not initialized" error
 * Solution: Call useNotifications() hook before using notifications
 * 
 * Issue: Backend can't send notifications
 * Solution: Check Firebase Admin SDK credentials and permissions
 * 
 * Issue: Android notifications not showing
 * Solution: Verify notification channel is created in app.json
 */

// ============================================================
// PRODUCTION CHECKLIST
// ============================================================

/**
 * Before launching to production:
 * 
 * ✅ Firebase Admin SDK installed
 * ✅ Service account key secured (not in Git)
 * ✅ FCM tokens stored in user database
 * ✅ Token update endpoint implemented
 * ✅ Notification handlers created for all types
 * ✅ Deep linking configured
 * ✅ Testing on real Android and iOS devices
 * ✅ Production build works with notifications
 * ✅ Backend script for sending notifications ready
 * ✅ Error handling and invalid token cleanup
 * ✅ Mobile notification privacy compliance (GDPR, CAN-SPAM)
 * ✅ Unsubscribe mechanism for users
 * ✅ Analytics tracking setup
 * ✅ Monitoring and alerting in place
 */

// ============================================================
// QUICK START: COPY-PASTE COMMANDS
// ============================================================

/**
 * Install Firebase messaging in your app:
 * cd clientapp && npm install @react-native-firebase/messaging
 * 
 * Install Firebase Admin in backend:
 * cd ArthwiseServices && npm install firebase-admin
 * 
 * Rebuild native code:
 * npx expo prebuild --clean
 * 
 * Run on Android:
 * npx expo run:android
 * 
 * Run on iOS:
 * npx expo run:ios
 */

// ============================================================
// KEY FILES TO CREATE/UPDATE
// ============================================================

/**
 * ✅ /clientapp/firebaseConfig.js - Updated with messaging
 * ✅ /clientapp/services/NotificationService.js - Created
 * ✅ /clientapp/services/NotificationHandlers.js - Created
 * ✅ /clientapp/hooks/useNotifications.js - Created
 * ✅ /clientapp/FIREBASE_MESSAGING_SETUP.md - Created
 * ✅ /clientapp/EXAMPLE_APP_INTEGRATION.md - Created
 * ✅ /ArthwiseServices/firebase-messaging-service.js - Created
 * ✅ /ArthwiseServices/.env - Create with credentials
 * 
 * Existing files to verify:
 * ✅ /clientapp/app.json - Check notification plugin
 * ✅ /clientapp/android/app/google-services.json
 * ✅ /clientapp/ios/GoogleService-Info.plist
 */

// ============================================================
// RECOMMENDED NOTIFICATION STRATEGY
// ============================================================

/**
 * For your use cases:
 * 
 * APP UPDATE:
 * - Send to users with appVersion < newVersion
 * - Include direct link to Play Store
 * - Set isForceUpdate: 'false' for optional updates
 * - Retry for 7 days
 * 
 * NEW CONTEST:
 * - Send to users who've participated before
 * - Include contest ID for deep linking
 * - High priority to show immediately
 * - Retry for 3 days
 * 
 * BEST TIME TO SEND:
 * - Update notifications: 12-1 PM or 8-9 PM (user-preferred times)
 * - Contest notifications: 9-10 AM (market open time)
 * - Always respect user timezone
 */

export const SETUP_CHECKLIST = 'All steps above';
