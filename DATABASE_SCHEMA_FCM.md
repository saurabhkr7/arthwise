/**
 * DATABASE SCHEMA FOR FCM
 * How to structure your database to track app versions and FCM tokens
 */

// ============================================================
// MONGODB / FIRESTORE SCHEMA
// ============================================================

/**
 * Collection: users
 * 
 * {
 *   _id: ObjectId("507f1f77bcf86cd799439011"),
 *   
 *   // User info
 *   email: "user@example.com",
 *   userId: "123456",
 *   createdAt: ISODate("2024-01-15T10:00:00Z"),
 *   
 *   // APP VERSION TRACKING (important for sending updates!)
 *   appVersion: "1.3.5",    // Current version they're running
 *   platform: "android",     // or "ios"
 *   lastAppVersionUpdate: ISODate("2024-04-10T14:30:00Z"),
 *   
 *   // NOTIFICATION SETTINGS
 *   notificationsEnabled: true,
 *   notificationPreferences: {
 *     app_updates: true,
 *     contests: true,
 *     promotions: false,
 *     news: true,
 *     reminders: true
 *   },
 *   
 *   // FCM TOKENS (one per device)
 *   fcmTokens: [
 *     {
 *       token: "ABC123DEF456GHI789...", // Firebase token
 *       platform: "android",             // Device platform
 *       appVersion: "1.3.5",            // App version on this device
 *       deviceName: "Samsung Galaxy S21",
 *       deviceOs: "Android 12",
 *       createdAt: ISODate("2024-03-20T09:15:00Z"),
 *       lastUsed: ISODate("2024-04-18T14:22:00Z"),
 *       isActive: true
 *     },
 *     {
 *       token: "XYZ789UVW012ABC345...",
 *       platform: "ios",
 *       appVersion: "1.3.4",
 *       deviceName: "iPhone 13",
 *       deviceOs: "iOS 16.4",
 *       createdAt: ISODate("2024-02-10T16:45:00Z"),
 *       lastUsed: ISODate("2024-04-15T10:00:00Z"),
 *       isActive: true
 *     }
 *   ],
 *   
 *   // NOTIFICATION LOG (track what was sent)
 *   notificationHistory: [
 *     {
 *       _id: ObjectId("607f191e810c19729de860ea"),
 *       type: "app_update",
 *       version: "1.4.0",
 *       sentAt: ISODate("2024-04-18T12:00:00Z"),
 *       status: "sent",
 *       clickedAt: ISODate("2024-04-18T12:15:00Z"),
 *       updatedToNewVersion: true
 *     },
 *     {
 *       _id: ObjectId("607f191e810c19729de860eb"),
 *       type: "contest",
 *       contestId: "507",
 *       sentAt: ISODate("2024-04-17T09:00:00Z"),
 *       status: "sent",
 *       clickedAt: ISODate("2024-04-17T09:30:00Z"),
 *       action: "joined_contest"
 *     }
 *   ]
 * }
 */

// ============================================================
// MONGODB QUERIES FOR SENDING NOTIFICATIONS
// ============================================================

/**
 * 1. FIND USERS WITH OLD APP VERSION:
 * 
 * db.users.find({
 *   appVersion: { $lt: "1.4.0" },
 *   notificationsEnabled: true,
 *   "fcmTokens.0": { $exists: true }  // Has at least one token
 * })
 * 
 * Result: Users with version < 1.4.0 who have at least one FCM token
 */

/**
 * 2. FIND ANDROID USERS WITH OLD VERSION:
 * 
 * db.users.find({
 *   platform: "android",
 *   appVersion: { $lt: "1.4.0" },
 *   notificationsEnabled: true,
 *   "fcmTokens.platform": "android"
 * })
 */

/**
 * 3. FIND iOS USERS WITH OLD VERSION:
 * 
 * db.users.find({
 *   platform: "ios",
 *   appVersion: { $lt: "1.4.0" },
 *   notificationsEnabled: true,
 *   "fcmTokens.platform": "ios"
 * })
 */

/**
 * 4. COUNT HOW MANY USERS WILL GET THE NOTIFICATION:
 * 
 * db.users.countDocuments({
 *   appVersion: { $lt: "1.4.0" },
 *   notificationsEnabled: true,
 *   "fcmTokens.0": { $exists: true }
 * })
 */

/**
 * 5. FIND USERS WHO HAVEN'T UPDATED AFTER 7 DAYS:
 * 
 * db.users.find({
 *   "notificationHistory": {
 *     $elemMatch: {
 *       type: "app_update",
 *       version: "1.4.0",
 *       updatedToNewVersion: false,
 *       sentAt: { $lt: ISODate("2024-04-11T00:00:00Z") }
 *     }
 *   }
 * })
 */

// ============================================================
// MONGODB UPDATES FOR MANAGING FCM
// ============================================================

/**
 * 1. STORE USER'S APP VERSION (when they login):
 * 
 * db.users.updateOne(
 *   { _id: userId },
 *   {
 *     $set: {
 *       appVersion: "1.3.5",
 *       lastAppVersionUpdate: new Date()
 *     }
 *   }
 * )
 */

/**
 * 2. ADD NEW FCM TOKEN (new device):
 * 
 * db.users.updateOne(
 *   { _id: userId },
 *   {
 *     $addToSet: {
 *       fcmTokens: {
 *         token: "NEW_TOKEN_ABC123...",
 *         platform: "android",
 *         appVersion: "1.3.5",
 *         createdAt: new Date(),
 *         lastUsed: new Date(),
 *         isActive: true
 *       }
 *     }
 *   }
 * )
 */

/**
 * 3. REMOVE INVALID/EXPIRED TOKEN:
 * 
 * db.users.updateOne(
 *   { _id: userId },
 *   {
 *     $pull: {
 *       fcmTokens: { token: "INVALID_TOKEN_XYZ789..." }
 *     }
 *   }
 * )
 */

/**
 * 4. UPDATE TOKEN LAST USED TIME:
 * 
 * db.users.updateOne(
 *   { _id: userId, "fcmTokens.token": tokenValue },
 *   {
 *     $set: {
 *       "fcmTokens.$.lastUsed": new Date()
 *     }
 *   }
 * )
 */

/**
 * 5. LOG NOTIFICATION EVENT:
 * 
 * db.users.updateOne(
 *   { _id: userId },
 *   {
 *     $push: {
 *       notificationHistory: {
 *         _id: new ObjectId(),
 *         type: "app_update",
 *         version: "1.4.0",
 *         sentAt: new Date(),
 *         status: "sent"
 *       }
 *     }
 *   }
 * )
 */

// ============================================================
// FIRESTORE EQUIVALENT
// ============================================================

/**
 * Firestore uses similar structure but different syntax:
 * 
 * Collection: users
 * Document: {userId}
 * Fields:
 * {
 *   email: "user@example.com",
 *   appVersion: "1.3.5",
 *   fcmTokens: [
 *     {
 *       token: "ABC123...",
 *       platform: "android",
 *       appVersion: "1.3.5",
 *       createdAt: Timestamp,
 *       lastUsed: Timestamp
 *     }
 *   ],
 *   notificationHistory: [
 *     {
 *       type: "app_update",
 *       version: "1.4.0",
 *       sentAt: Timestamp,
 *       status: "sent"
 *     }
 *   ]
 * }
 * 
 * // Query for old versions (similar to MongoDB)
 * db.collection('users')
 *   .where('appVersion', '<', '1.4.0')
 *   .where('notificationsEnabled', '==', true)
 *   .get()
 */

// ============================================================
// BACKEND CODE TO HANDLE DATABASE
// ============================================================

/**
 * Node.js Express Example with MongoDB:
 */

const express = require('express');
const { MongoClient } = require('mongodb');
const admin = require('firebase-admin');

const app = express();
const mongoUrl = 'mongodb://localhost:27017';
const dbName = 'arthwise';
const collName = 'users';

let mongoClient;
let db;

// Connect to MongoDB
async function connectDB() {
  mongoClient = new MongoClient(mongoUrl);
  await mongoClient.connect();
  db = mongoClient.db(dbName);
  console.log('✅ Connected to MongoDB');
}

// UPDATE APP VERSION WHEN USER LOGS IN
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password, appVersion } = req.body;

    // ... authentication logic ...

    // Store app version
    await db.collection(collName).updateOne(
      { email },
      {
        $set: {
          appVersion,
          lastAppVersionUpdate: new Date()
        }
      }
    );

    res.json({ success: true, token: 'JWT_TOKEN_HERE' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// ADD/UPDATE FCM TOKEN
app.post('/api/user/update-fcm-token', async (req, res) => {
  try {
    const { userId, fcmToken, platform, appVersion } = req.body;

    await db.collection(collName).updateOne(
      { _id: userId },
      {
        $addToSet: {
          fcmTokens: {
            token: fcmToken,
            platform,
            appVersion,
            createdAt: new Date(),
            lastUsed: new Date(),
            isActive: true
          }
        }
      }
    );

    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// GET USERS WITH OLD VERSION & SEND NOTIFICATION
app.post('/api/admin/notify-update', async (req, res) => {
  try {
    const { newVersion } = req.body;
    const messaging = admin.messaging();

    // Find users with old version
    const oldUsers = await db.collection(collName).find({
      appVersion: { $lt: newVersion },
      notificationsEnabled: true,
      'fcmTokens.0': { $exists: true }
    }).toArray();

    console.log(`Found ${oldUsers.length} users with old version`);

    let sent = 0;
    let failed = 0;

    for (const user of oldUsers) {
      for (const fcmData of user.fcmTokens || []) {
        try {
          await messaging.send({
            notification: {
              title: '🚀 Update Available',
              body: `Arthwise v${newVersion} - Lighter & Faster!`
            },
            data: {
              notificationType: 'app_update',
              newVersion,
              playStoreUrl: 'https://play.google.com/store/apps/details?id=com.arthwise',
              appStoreUrl: 'https://apps.apple.com/app/arthwise/...',
              isForceUpdate: 'false'
            },
            token: fcmData.token
          });
          sent++;

          // Log notification sent
          await db.collection(collName).updateOne(
            { _id: user._id },
            {
              $push: {
                notificationHistory: {
                  type: 'app_update',
                  version: newVersion,
                  sentAt: new Date(),
                  status: 'sent'
                }
              }
            }
          );
        } catch (error) {
          failed++;
          
          // Remove invalid token
          if (error.code === 'messaging/invalid-registration-token') {
            await db.collection(collName).updateOne(
              { _id: user._id },
              { $pull: { fcmTokens: { token: fcmData.token } } }
            );
          }
        }
      }
    }

    res.json({ success: true, sent, failed });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Start server
connectDB().then(() => {
  app.listen(3000, () => console.log('Server running on port 3000'));
});

// ============================================================
// MIGRATION: Adding FCM to Existing Users
// ============================================================

/**
 * If you have existing users without fcmTokens:
 * 
 * db.users.updateMany(
 *   { fcmTokens: { $exists: false } },
 *   {
 *     $set: {
 *       fcmTokens: [],
 *       notificationsEnabled: true,
 *       notificationHistory: []
 *     }
 *   }
 * )
 */

// ============================================================
// CLEANUP: Remove Old/Inactive Tokens
// ============================================================

/**
 * Periodically remove tokens not used in 90 days:
 * 
 * db.users.updateMany(
 *   {
 *     'fcmTokens.lastUsed': {
 *       $lt: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000)
 *     }
 *   },
 *   {
 *     $pull: {
 *       fcmTokens: {
 *         lastUsed: {
 *           $lt: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000)
 *         }
 *       }
 *     }
 *   }
 * )
 */

export const DATABASE_SCHEMA = 'See above for complete schema';
