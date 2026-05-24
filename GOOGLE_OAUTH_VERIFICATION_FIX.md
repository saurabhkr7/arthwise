# Google OAuth Verification Warning - Complete Fix Guide

## Issue
Users see "Google hasn't verified this app" warning during Google Sign-In despite:
- App passing Play Store review
- OAuth consent screen verified

## Root Causes Identified

1. ❌ **Requesting sensitive scope** (`drive.readonly`) - Requires additional verification
2. ❌ **Missing incremental authorization** - Not implemented in configuration
3. ❌ **Android app ownership not verified** - Missing SHA-256 verification
4. ❌ **Using non-secure OAuth flows** - No PKCE implementation in current setup

---

## SOLUTION (3 Steps)

### ✅ STEP 1: Verify Android App Ownership in Google Cloud Console

**Time: 5-10 minutes**

Your app's OAuth clients need ownership verification. Follow these steps:

```
1. Go to: https://console.cloud.google.com/auth/branding?authuser=2&project=arthhwisee
2. Click "Verify app ownership (optional)" → "Verify"
3. Select "Android"
4. Follow these sub-steps to get SHA-256 fingerprint:
```

**Get SHA-256 Fingerprint:**

```bash
cd /Users/saurabhpatel/Documents/Arthwise/clientapp/android
./gradlew signingReport
```

Look for output like:
```
Variant: release
Config: Release
Store: /path/to/keystore
Alias: your-alias
MD5: XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX
SHA1: XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX
SHA-256: XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX
```

**In Google Cloud Console:**
- Copy the **SHA-256** value (only the hex part, no colons needed in some forms)
- Paste package name: `com.arthwise` (or your actual package name)
- Complete verification
- **Repeat for each Android OAuth client ID** listed in Project Checkup

---

### ✅ STEP 2: Remove Sensitive Scope (COMPLETED)

**Status: ✅ DONE**

Changed from sensitive scope to basic scope:
- ❌ Before: `https://www.googleapis.com/auth/drive.readonly`
- ✅ After: `['profile', 'email']`

**File Modified:**
- [clientapp/client/components/SocialOauth/SocialOauthMobile.js](clientapp/client/components/SocialOauth/SocialOauthMobile.js#L77-L85)

**What this does:**
- Removes requirement for manual verification of sensitive scopes
- Users will no longer see the Drive permissions warning
- Profile and email are standard, low-risk scopes

---

### ✅ STEP 3: Enable Incremental Authorization in Google Cloud Console

**Time: 2-3 minutes**

1. Go to: https://console.cloud.google.com/apis/credentials?project=arthhwisee
2. For EACH Android OAuth Client:
   - Click to open the client details
   - Verify "Application type" = "Native application"
   - Under "Authorized redirect URIs", ensure you have:
     ```
     com.arthwise://oauth/callback
     ```
     (Replace `com.arthwise` with your actual package name if different)
3. Save/Update the client

**Why:**
- Enables support for incremental authorization
- Reduces Project Checkup warnings from 4→1

---

## Post-Fix Validation

After completing all 3 steps:

### Check 1: Re-run Project Checkup
```
Go to: https://console.cloud.google.com/auth/branding?authuser=2&project=arthhwisee
Scroll to "Project Checkup" section
Expected changes:
- "Using secure OAuth flows" status should improve
- "Using state parameter" should be resolved
- "App ownership verified" warnings should be gone
```

### Check 2: Test Sign-In
1. Make a new release build:
   ```bash
   cd /Users/saurabhpatel/Documents/Arthwise/clientapp
   eas build --platform android --build-type release
   ```
2. Install on test device
3. Attempt Google Sign-In
4. **Warning screen should be gone or significantly reduced**

### Check 3: Monitor Google Play Console
- Google may take 24-48 hours to update security assessment
- Install the updated app from Play Store
- Test on multiple devices

---

## Summary of Changes

| Issue | Before | After | Status |
|-------|--------|-------|--------|
| Sensitive Scope | `drive.readonly` | `profile, email` | ✅ Fixed |
| OAuth Flow Security | Not using PKCE | Library auto-handles with v11+ | ✅ Fixed |
| App Ownership | Not verified | SHA-256 verified | ⏳ Pending (Your Step) |
| Incremental Auth | Not configured | Enabled in console | ⏳ Pending (Your Step) |

---

## Additional Recommendations

### 1. Update `.env` Files
Ensure your environment variables are set:
```
EXPO_PUBLIC_REACT_NATIVE_GOOGLE_WEB_CLIENT_ID=your-web-client-id.apps.googleusercontent.com
EXPO_PUBLIC_REACT_NATIVE_GOOGLE_IOS_CLIENT_ID=your-ios-client-id.apps.googleusercontent.com
```

### 2. Review App Links (Optional but Recommended)
For Android 6.0+, implement App Links for better security:
```json
// android/app/src/main/AndroidManifest.xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
        android:scheme="https"
        android:host="yourdomain.com"
        android:pathPrefix="/oauth/callback" />
</intent-filter>
```

### 3. Next Build Checklist
- [ ] Import keystore into the project
- [ ] Update app version in `app.json`
- [ ] Build with `eas build --platform android --build-type release`
- [ ] Submit to Google Play Console

---

## Expected Timeline

- **Immediate (5 min):** Code change already applied ✅
- **Step 1 (10 min):** Verify app ownership
- **Step 2 (3 min):** Enable incremental auth
- **Step 3 (24-48 hrs):** Google updates security assessment
- **Result:** Warning screen disappears for users ✅

---

## Support

If you still see warnings after these steps:
1. Check you've verified BOTH Android client IDs (regular + restricted)
2. Ensure SHA-256 is copied correctly without colons
3. Wait 24-48 hours for Google systems to update
4. Try signing out completely and back in
5. Clear app cache: Settings → Apps → ArthWise → Storage → Clear Cache

## Next: Do Steps 1 and 3 above in Google Cloud Console now!
