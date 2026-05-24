/**
 * HOW TO SEND NOTIFICATIONS TO OLD VERSION USERS
 * Practical guide: Telling users about new app update with Play Store/App Store links
 * 
 * This is exactly what you asked for - sending update notifications
 * to users with older versions and including the store links
 */

// ============================================================
// SCENARIO: YOU RELEASED VERSION 1.4.0
// ============================================================

/**
 * Your app currently has:
 * - Version 1.3.5 in production (users are running this)
 * - New version 1.4.0 released (uploaded to Play Store/App Store)
 * - 50% lighter, faster, new features
 * 
 * GOAL: Notify all 1.3.5 users about 1.4.0 with direct store links
 */

// ============================================================
// STEP 1: TRACK APP VERSIONS IN DATABASE
// ============================================================

/**
 * Your backend must track which app version each user has.
 * 
 * When user logs in, send their current version:
 * 
 * POST /api/user/login
 * {
 *   email: "user@example.com",
 *   password: "xxx",
 *   appVersion: "1.3.5"  // Send from app.json version
 * }
 * 
 * Backend stores this:
 * users collection: {
 *   _id: "user123",
 *   email: "user@example.com",
 *   appVersion: "1.3.5",
 *   fcmTokens: [
 *     {
 *       token: "ABC123...",
 *       platform: "android",
 *       appVersion: "1.3.5",
 *       lastUsed: "2024-04-18"
 *     }
 *   ]
 * }
 */

// ============================================================
// STEP 2: CREATE BACKEND FUNCTION TO SEND UPDATE NOTIFICATION
// ============================================================

/**
 * In your ArthwiseServices/backend:
 * 
 * const admin = require('firebase-admin');
 * const db = admin.firestore();
 * const messaging = admin.messaging();
 * 
 * async function notifyOldVersionUsers(newVersion) {
 *   console.log(`\n📤 Sending update notification for version ${newVersion}...\n`);
 * 
 *   try {
 *     // Find ALL users with version < newVersion
 *     const usersSnapshot = await db.collection('users')
 *       .where('appVersion', '<', newVersion)
 *       .get();
 * 
 *     console.log(`Found ${usersSnapshot.docs.length} users with older version`);
 * 
 *     let sentCount = 0;
 *     let failedCount = 0;
 * 
 *     for (const userDoc of usersSnapshot.docs) {
 *       const user = userDoc.data();
 *       const fcmTokens = user.fcmTokens || [];
 * 
 *       if (!fcmTokens.length) {
 *         console.log(`⚠️  Skip: ${user.email} has no FCM tokens`);
 *         continue;
 *       }
 * 
 *       // Send to all their devices
 *       for (const fcmData of fcmTokens) {
 *         const message = {
 *           notification: {
 *             title: '🚀 Major Update Available!',
 *             body: 'Arthwise v' + newVersion + ' - Lighter & Faster!',
 *           },
 *           data: {
 *             notificationType: 'app_update',
 *             newVersion: newVersion,
 *             oldVersion: user.appVersion,
 *             // PLAY STORE LINK
 *             playStoreUrl: 'https://play.google.com/store/apps/details?id=com.arthwise',
 *             // APP STORE LINK  
 *             appStoreUrl: 'https://apps.apple.com/app/arthwise/id1234567890',
 *             features: JSON.stringify([
 *               '50% smaller file size',
 *               'Significantly faster performance',
 *               'New portfolio analysis tools',
 *               'Enhanced security',
 *               'Better user experience'
 *             ]),
 *             isForceUpdate: 'false', // User can postpone
 *           },
 *           token: fcmData.token,
 *           android: {
 *             priority: 'high',
 *             ttl: 604800, // 7 days retry
 *             notification: {
 *               sound: 'default',
 *               channelId: 'app_updates',
 *             },
 *           },
 *         };
 * 
 *         try {
 *           await messaging.send(message);
 *           sentCount++;
 *           console.log(`✅ Sent to ${user.email} (${fcmData.platform})`);
 *         } catch (error) {
 *           failedCount++;
 *           console.error(`❌ Failed for ${user.email}:`, error.message);
 * 
 *           // Remove invalid tokens
 *           if (error.code === 'messaging/invalid-registration-token') {
 *             await db.collection('users').doc(userDoc.id).update({
 *               fcmTokens: admin.firestore.FieldValue.arrayRemove(fcmData)
 *             });
 *           }
 *         }
 *       }
 *     }
 * 
 *     console.log(`\n✅ COMPLETE: Sent ${sentCount}, Failed ${failedCount}\n`);
 *     return { sent: sentCount, failed: failedCount };
 *   } catch (error) {
 *     console.error('❌ Error:', error);
 *     throw error;
 *   }
 * }
 */

// ============================================================
// STEP 3: EXPORT AND USE THE FUNCTION
// ============================================================

/**
 * Then use it from your backend:
 * 
 * // In your index.js or app.js
 * const { notifyOldVersionUsers } = require('./firebase-messaging-service');
 * 
 * // Call this after releasing 1.4.0 to stores
 * app.post('/api/admin/notify-update', async (req, res) => {
 *   try {
 *     const { newVersion } = req.body;
 *     const result = await notifyOldVersionUsers(newVersion);
 *     res.json({ success: true, ...result });
 *   } catch (error) {
 *     res.status(500).json({ error: error.message });
 *   }
 * });
 * 
 * // Or run from command line
 * // node -e "require('./firebase-messaging-service').notifyOldVersionUsers('1.4.0')"
 */

// ============================================================
// STEP 4: WHAT THE USER SEES
// ============================================================

/**
 * When notification arrives:
 * 
 * ANDROID NOTIFICATION TRAY:
 * ┌─────────────────────────────────┐
 * │ 🚀 Major Update Available!       │
 * │ Arthwise v1.4.0 - Lighter...     │
 * │ Arthwise                         │
 * └─────────────────────────────────┘
 * 
 * Tap notification -> App handles it
 * 
 * IN-APP ALERT (if app open):
 * ┌─────────────────────────────────┐
 * │ 🚀 Major Update Available!      │
 * │                                 │
 * │ Arthwise v1.4.0 - Lighter!      │
 * │                                 │
 * │ [Update Now]  [Later]           │
 * └─────────────────────────────────┘
 * 
 * User taps "Update Now" -> Opens Play Store/App Store link
 * User taps "Later" -> Dismisses, tries again in a week
 */

// ============================================================
// STEP 5: EXACT API SPECIFICATION
// ============================================================

/**
 * POST /api/admin/notify-update
 * 
 * Request Body:
 * {
 *   "newVersion": "1.4.0"
 * }
 * 
 * Response on Success:
 * {
 *   "success": true,
 *   "sent": 2345,      // Number of notifications sent
 *   "failed": 12       // Invalid tokens, etc.
 * }
 * 
 * Response on Failure:
 * {
 *   "error": "Firebase error message"
 * }
 */

// ============================================================
// STEP 6: FRONTEND HANDLER - WHAT HAPPENS WHEN USER TAPS
// ============================================================

/**
 * In your clientapp/services/NotificationHandlers.js:
 * 
 * export const handleAppUpdateNotification = (remoteMessage) => {
 *   const { notification, data } = remoteMessage;
 *   const {
 *     newVersion,
 *     playStoreUrl,
 *     appStoreUrl,
 *     features,
 *     isForceUpdate,
 *   } = data;
 * 
 *   // Parse features if it's JSON string
 *   let featuresText = features;
 *   try {
 *     const featuresList = JSON.parse(features);
 *     featuresText = featuresList.join('\n');
 *   } catch (e) {
 *     // Keep original if not JSON
 *   }
 * 
 *   const isForce = isForceUpdate === 'true';
 * 
 *   const buttons = [
 *     {
 *       text: 'Update Now',
 *       onPress: async () => {
 *         try {
 *           const storeUrl = Platform.OS === 'android' 
 *             ? playStoreUrl 
 *             : appStoreUrl;
 *           
 *           if (storeUrl) {
 *             await Linking.openURL(storeUrl);
 *             console.log('✅ Opened app store');
 *           }
 *         } catch (error) {
 *           Alert.alert('Error', 'Could not open app store');
 *         }
 *       },
 *     },
 *   ];
 * 
 *   // Only show "Later" if not a forced update
 *   if (!isForce) {
 *     buttons.push({
 *       text: 'Later',
 *       onPress: () => console.log('Update postponed'),
 *       style: 'cancel',
 *     });
 *   }
 * 
 *   Alert.alert(
 *     `Update v${newVersion} Available`,
 *     featuresText,
 *     buttons,
 *     { cancelable: !isForce }
 *   );
 * };
 */

// ============================================================
// STEP 7: STEP-BY-STEP TIMELINE
// ============================================================

/**
 * FRIDAY - Release to stores:
 * 1. Build release version 1.4.0
 * 2. Upload to Google Play Console (Internal Testing first)
 * 3. Upload to App Store TestFlight
 * 4. Test both builds
 * 5. Submit to production
 * 
 * SATURDAY - Wait for approval:
 * 1. Google Play: Usually ~2-4 hours review
 * 2. App Store: Usually ~24 hours review (sometimes faster)
 * 3. Once approved, versions will appear in Play Store
 * 
 * SUNDAY - Notify users (wait for store indexing):
 * 1. Verify version 1.4.0 is live in both stores
 * 2. Test the store links yourself
 * 3. Call your backend API: POST /api/admin/notify-update { "newVersion": "1.4.0" }
 * 4. Monitor notification delivery in Firebase Console
 * 5. Check that store links work from devices
 * 
 * EMAILS/COMMS (Send to customer support):
 * "New version available to all users, check dashboard"
 */

// ============================================================
// STEP 8: CHECKING THINGS ARE WORKING
// ============================================================

/**
 * After calling /api/admin/notify-update:
 * 
 * 1. Firebase Console:
 *    - Go to Cloud Messaging
 *    - See impressions count increase
 *    - Click campaign to see stats
 * 
 * 2. Your Database:
 *    - Query users with appVersion < 1.4.0
 *    - Should be many (those who haven't updated yet)
 * 
 * 3. Test Devices:
 *    - Keep an old version device
 *    - Force close the app
 *    - Open Settings > Apps > Arthwise > Force Stop
 *    - Wait 5 minutes
 *    - Check notification tray
 *    - You should see the update notification
 *    - Tap it, should open Play Store
 * 
 * 4. Production Metrics:
 *    - After 1-2 hours, check Play Store Console
 *    - "Installs" should increase significantly
 * 
 * 5. Error Monitoring:
 *    - Invalid token: Remove from database
 *    - User opted out: They won't receive (respects settings)
 *    - Network error: Automatically retried for 7 days
 */

// ============================================================
// OPTIONAL: USE FIREBASE CONSOLE DIRECTLY
// ============================================================

/**
 * Instead of backend code, you can also use Firebase Console:
 * 
 * 1. Firebase Console > Messaging > New Campaign
 * 2. Select "Firebase Apps"
 * 3. Select "Android" or "iOS" or both
 * 
 * 4. Notification Details:
 *    Title: "🚀 Major Update Available!"
 *    Body: "Arthwise v1.4.0 - Lighter & Faster!"
 * 
 * 5. Custom Data (Key-Value):
 *    Key: "notificationType"    Value: "app_update"
 *    Key: "newVersion"          Value: "1.4.0"
 *    Key: "playStoreUrl"        Value: "https://play.google.com/store/apps/details?id=com.arthwise"
 *    Key: "appStoreUrl"         Value: "https://apps.apple.com/app/arthwise/..."
 *    Key: "isForceUpdate"       Value: "false"
 * 
 * 6. Target Users:
 *    - Option A: All users
 *    - Option B: Custom targeting (by app version if supported)
 * 
 * 7. Schedule:
 *    - Send now or schedule for later
 * 
 * 8. Send!
 */

// ============================================================
// REAL EXAMPLE: COMPLETE WORKING CODE
// ============================================================

/**
 * MINIMAL COPY-PASTE BACKEND CODE:
 * 
 * Save as: ArthwiseServices/send-update-notification.js
 */

const admin = require('firebase-admin');
const serviceAccount = require('./serviceAccountKey.json');
const db = require('better-sqlite3')('user_data.db'); // or your DB

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
});

const messaging = admin.messaging();

async function sendUpdateNotification(newVersion, oldMaxVersion) {
  console.log(`📤 Sending update notification for version ${newVersion}...`);

  // Example: Get users from your database
  // Adjust query based on your database
  const users = [
    {
      email: 'user1@example.com',
      appVersion: '1.3.5',
      fcmTokens: [
        { token: 'ABC123...', platform: 'android' },
      ]
    },
    // ... more users
  ];

  let success = 0;
  let failed = 0;

  for (const user of users) {
    if (user.appVersion >= newVersion) continue; // Already new version

    for (const fcm of user.fcmTokens) {
      try {
        await messaging.send({
          notification: {
            title: '🚀 Update Available',
            body: `Arthwise v${newVersion} - Lighter & Faster!`,
          },
          data: {
            notificationType: 'app_update',
            newVersion,
            playStoreUrl: 'https://play.google.com/store/apps/details?id=com.arthwise',
            appStoreUrl: 'https://apps.apple.com/app/arthwise/id1234567890',
            isForceUpdate: 'false',
          },
          token: fcm.token,
        });
        success++;
        console.log(`✅ ${user.email}`);
      } catch (error) {
        failed++;
        console.error(`❌ ${user.email}: ${error.message}`);
      }
    }
  }

  console.log(`\n✅ DONE: ${success} sent, ${failed} failed`);
}

// Run it
sendUpdateNotification('1.4.0').catch(console.error);
