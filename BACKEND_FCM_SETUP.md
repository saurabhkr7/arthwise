/**
 * 🔧 BACKEND SETUP: FCM Token Storage & Notification Endpoints
 * 
 * Integration guide for your ArthwiseServices Express backend
 */

// ============================================================
// STEP 1: INSTALL DEPENDENCIES
// ============================================================

/**
 * In /ArthwiseServices directory:
 * 
 * npm install firebase-admin
 * npm install axios (if not already installed)
 * 
 * Verify in package.json after install:
 * "firebase-admin": "^12.x.x"
 * 
 */

// ============================================================
// STEP 2: GET FIREBASE SERVICE ACCOUNT KEY
// ============================================================

/**
 * This is CRITICAL - without this, backend can't send notifications
 * 
 * 1. Go to: https://console.firebase.google.com
 * 2. Select your Arthwise project
 * 3. Go to: Project Settings (⚙️ icon bottom left)
 * 4. Click "Service Accounts" tab
 * 5. Click "Generate New Private Key"
 * 6. Save the JSON file
 * 
 * SECURITY:
 * - Save as: /ArthwiseServices/serviceAccountKey.json
 * - Add to .gitignore: echo "serviceAccountKey.json" >> .gitignore
 * - NEVER commit this to Git!
 * - NEVER share this key!
 * 
 */

// ============================================================
// STEP 3: CREATE FCM ROUTES FILE
// ============================================================

/**
 * Create new file: /ArthwiseServices/routes/fcmRoutes.js
 * 
 * Copy code from below section "FCM_ROUTES_CODE"
 */

// ============================================================
// STEP 4: CREATE FCM SERVICE FILE
// ============================================================

/**
 * Create new file: /ArthwiseServices/services/notificationService.js
 * 
 * Copy code from below section "NOTIFICATION_SERVICE_CODE"
 */

// ============================================================
// STEP 5: UPDATE index.js TO INCLUDE FCM ROUTES
// ============================================================

/**
 * In /ArthwiseServices/index.js, add these imports at the top:
 * 
 * import { fcmRoutes } from './routes/fcmRoutes.js';
 * 
 * Then add this line after other route definitions (around line 14):
 * 
 * // Firebase Cloud Messaging (FCM) routes
 * fcmRoutes(app);
 * 
 * Full example:
 * ---
 * import { newsFeedRoute } from './nse_service/newsFeed/index.js';
 * import { fcmRoutes } from './routes/fcmRoutes.js';  // ADD THIS
 * 
 * const app = express();
 * ...
 * DashboardData(app);
 * newsFeedRoute(app);
 * fcmRoutes(app);  // ADD THIS
 * ---
 */

// ============================================================
// STEP 6: CREATE .env FILE WITH FIREBASE CONFIG
// ============================================================

/**
 * Create file: /ArthwiseServices/.env
 * 
 * Add your Firebase credentials:
 * 
 * FIREBASE_PROJECT_ID=arthwise-xxxxx
 * FIREBASE_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----
 * \n... (your long key)
 * -----END PRIVATE KEY-----\n
 * FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxxxx@arthwise-xxxxx.iam.gserviceaccount.com
 * 
 * Or load from file:
 * FIREBASE_SERVICE_ACCOUNT_KEY=./serviceAccountKey.json
 * 
 * 
 * NOTE: In .env format, newlines should be literal \n not actual newlines
 * 
 * Easy way to generate this from your JSON file:
 * cat serviceAccountKey.json | jq -r '.private_key' 
 *                                    ↑
 * This outputs the key in the right format for .env
 */

// ============================================================
// FCM_ROUTES_CODE - Copy this into routes/fcmRoutes.js
// ============================================================

/**
 * File: /ArthwiseServices/routes/fcmRoutes.js
 */

import { notificationService } from '../services/notificationService.js';

export const fcmRoutes = (app) => {
  // Store/update FCM token for a user
  app.post('/api/fcm/update-token', async (req, res) => {
    try {
      const { userId, fcmToken, platform, appVersion } = req.body;

      // Validate input
      if (!userId || !fcmToken) {
        return res.status(400).json({
          error: 'userId and fcmToken are required'
        });
      }

      // Call notification service to store token
      const result = await notificationService.storeFCMToken(
        userId,
        fcmToken,
        platform || 'unknown',
        appVersion || 'unknown'
      );

      res.json({
        success: true,
        message: 'FCM token updated',
        userId
      });
    } catch (error) {
      console.error('❌ Error updating FCM token:', error);
      res.status(500).json({
        error: 'Failed to update FCM token',
        message: error.message
      });
    }
  });

  // Admin endpoint: Send app update notification
  app.post('/api/admin/notify-update', async (req, res) => {
    try {
      // TODO: Add authentication check for admin user
      // if (!isAdmin(req.user)) return res.status(403).json({ error: 'Unauthorized' });

      const { newVersion } = req.body;

      if (!newVersion) {
        return res.status(400).json({
          error: 'newVersion is required'
        });
      }

      const result = await notificationService.sendAppUpdateNotification(newVersion);

      res.json({
        success: true,
        ...result
      });
    } catch (error) {
      console.error('❌ Error sending app update notification:', error);
      res.status(500).json({
        error: 'Failed to send notification',
        message: error.message
      });
    }
  });

  // Test endpoint: Send test notification to single device
  app.post('/api/fcm/send-test', async (req, res) => {
    try {
      const { fcmToken, title, body } = req.body;

      if (!fcmToken) {
        return res.status(400).json({
          error: 'fcmToken is required'
        });
      }

      const result = await notificationService.sendTestNotification(
        fcmToken,
        title || 'Test Notification',
        body || 'This is a test notification'
      );

      res.json({
        success: true,
        message: 'Test notification sent',
        ...result
      });
    } catch (error) {
      console.error('❌ Error sending test notification:', error);
      res.status(500).json({
        error: 'Failed to send test notification',
        message: error.message
      });
    }
  });

  console.log('✅ FCM routes registered');
};

// ============================================================
// NOTIFICATION_SERVICE_CODE - Copy this into services/notificationService.js
// ============================================================

/**
 * File: /ArthwiseServices/services/notificationService.js
 */

import admin from 'firebase-admin';
import fs from 'fs';

// Initialize Firebase Admin SDK
let initialized = false;

const initializeFirebase = () => {
  if (initialized) return;

  try {
    // Try to load from serviceAccountKey.json file first
    if (fs.existsSync('./serviceAccountKey.json')) {
      const serviceAccount = JSON.parse(
        fs.readFileSync('./serviceAccountKey.json', 'utf8')
      );
      admin.initializeApp({
        credential: admin.credential.cert(serviceAccount),
      });
      console.log('✅ Firebase initialized from serviceAccountKey.json');
    } else {
      // Fallback to environment variables
      const serviceAccount = {
        projectId: process.env.FIREBASE_PROJECT_ID,
        privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
        clientEmail: process.env.FIREBASE_CLIENT_EMAIL,
      };

      admin.initializeApp({
        credential: admin.credential.cert(serviceAccount),
      });
      console.log('✅ Firebase initialized from environment variables');
    }

    initialized = true;
  } catch (error) {
    console.error('❌ Firebase initialization error:', error);
    throw error;
  }
};

// Store FCM token in database
export const notificationService = {
  async storeFCMToken(userId, fcmToken, platform = 'unknown', appVersion = 'unknown') {
    try {
      // TODO: Implement database storage
      // For now, this is a placeholder
      // You need to update your user document in database:
      // db.users.updateOne(
      //   { _id: userId },
      //   {
      //     $addToSet: {
      //       fcmTokens: {
      //         token: fcmToken,
      //         platform,
      //         appVersion,
      //         createdAt: new Date()
      //       }
      //     }
      //   }
      // )

      console.log(`✅ FCM token stored for user ${userId}`);
      return {
        success: true,
        userId,
        platform,
        appVersion
      };
    } catch (error) {
      console.error('Error storing FCM token:', error);
      throw error;
    }
  },

  async sendAppUpdateNotification(newVersion) {
    try {
      initializeFirebase();
      const messaging = admin.messaging();

      // TODO: Query your database for users with older version
      // const users = await db.users.find({ appVersion: { $lt: newVersion } });

      // This is a test implementation
      console.log(`📤 Would send app update notification for version ${newVersion}`);

      return {
        sent: 0,
        failed: 0,
        message: 'Database integration needed to retrieve users'
      };
    } catch (error) {
      console.error('Error sending app update notification:', error);
      throw error;
    }
  },

  async sendTestNotification(fcmToken, title, body) {
    try {
      initializeFirebase();
      const messaging = admin.messaging();

      const message = {
        notification: {
          title,
          body
        },
        data: {
          notificationType: 'test'
        },
        token: fcmToken,
        android: {
          priority: 'high'
        }
      };

      const response = await messaging.send(message);
      console.log('✅ Test notification sent:', response);

      return {
        success: true,
        messageId: response
      };
    } catch (error) {
      console.error('Error sending test notification:', error);
      throw error;
    }
  }
};

// ============================================================
// STEP 7: UPDATE APP.JS TO SEND TOKEN TO BACKEND
// ============================================================

/**
 * In your clientapp App.js, add this in the useEffect:
 * 
 * useEffect(() => {
 *   if (fcmToken && isLoggedIn && userId) {
 *     // Send FCM token to backend
 *     fetch('https://your-backend.com/api/fcm/update-token', {
 *       method: 'POST',
 *       headers: {
 *         'Content-Type': 'application/json',
 *       },
 *       body: JSON.stringify({
 *         userId,
 *         fcmToken,
 *         platform: Platform.OS,
 *         appVersion: '1.3.5'
 *       })
 *     })
 *     .then(r => r.json())
 *     .then(data => console.log('✅ Token sent to backend:', data))
 *     .catch(e => console.error('❌ Error sending token:', e));
 *   }
 * }, [fcmToken, isLoggedIn, userId]);
 */

// ============================================================
// TESTING THE ENDPOINTS
// ============================================================

/**
 * After everything is set up, you can test:
 * 
 * 1. Update FCM token endpoint:
 *    curl -X POST http://localhost:3000/api/fcm/update-token \
 *      -H "Content-Type: application/json" \
 *      -d '{
 *        "userId": "user123",
 *        "fcmToken": "ABC123...",
 *        "platform": "android",
 *        "appVersion": "1.3.5"
 *      }'
 * 
 * 2. Send test notification:
 *    curl -X POST http://localhost:3000/api/fcm/send-test \
 *      -H "Content-Type: application/json" \
 *      -d '{
 *        "fcmToken": "ABC123...",
 *        "title": "Test Title",
 *        "body": "Test body"
 *      }'
 * 
 * 3. Send app update notification:
 *    curl -X POST http://localhost:3000/api/admin/notify-update \
 *      -H "Content-Type: application/json" \
 *      -d '{"newVersion": "1.4.0"}'
 */

// ============================================================
// DATABASE SCHEMA YOU NEED
// ============================================================

/**
 * Your database needs to store:
 * 
 * users collection:
 * {
 *   _id: "user123",
 *   email: "user@example.com",
 *   appVersion: "1.3.5",  // V Important for sending updates!
 *   fcmTokens: [
 *     {
 *       token: "ABC123...",
 *       platform: "android",
 *       appVersion: "1.3.5",
 *       createdAt: ISODate("2024-04-18T10:00:00Z")
 *     }
 *   ]
 * }
 * 
 * See: DATABASE_SCHEMA_FCM.md for complete schema
 */

export const BACKEND_SETUP = 'See steps above';
