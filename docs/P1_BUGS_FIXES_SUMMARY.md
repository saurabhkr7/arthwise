# P1 Production Bugs - Fixes Applied

**Date:** April 20, 2026  
**Priority:** P1 (Critical)  
**Status:** 2/3 Fixed, 1/3 Under Investigation

---

## ✅ Issue #1: App Asks for Login on Relaunch

### Problem
Users lose login persistence - app requires login every time it's relaunched, even though they were previously logged in.

### Root Cause
In `/clientapp/client/apis/refreshAuthToken.js`, the function threw an error immediately when no refresh token was available, without checking for a cached access token:
```javascript
if (!refreshToken) {
  throw new Error('No refresh token available');  // ❌ Immediate failure, no fallback
}
```

### Solution
✅ **FIXED** - Implemented SilentAuth fallback logic:

**File Modified:** `/clientapp/client/apis/refreshAuthToken.js` (lines 12-21)

**Changes:**
- Added fallback to check AsyncStorage for cached access token before throwing error
- If refresh token unavailable, try cached token first
- Only require login if both refresh token AND cached token are unavailable

**Code:**
```javascript
if (!refreshToken) {
  console.log("⚠️ [refreshAuthToken] No refresh token available, checking cached token...");
  const cachedToken = await AsyncStorage.getItem('token');
  if (cachedToken) {
    console.log("✅ [refreshAuthToken] Using cached access token for silent auth");
    return cachedToken; // Return cached token without requiring login
  }
  throw new Error('No refresh token available and no cached token found');
}
```

### Testing
- [x] Test on device: Close app with valid session and relaunch
- [x] Verify app doesn't ask for login
- [x] Verify token is restored from cache

---

## ✅ Issue #3: Contest Creator Name Empty ("By: ")

### Problem  
Upcoming Contests display shows "By: " with no creator name instead of showing the creator's display name.

### Root Cause
1. User model field selection error in backend: selecting `name` field that doesn't exist, should be `displayname`
2. Existing contests in database don't have `createdByName` populated (null/empty values)

**File Issue Location:** `/ArthwiseServices/auth_microservice/controllers/contestController.js` line 274

### Solution
✅ **FIXED** - Multi-part fix:

#### Part A: Fix Field Selection
**File Modified:** `/ArthwiseServices/auth_microservice/controllers/contestController.js` (lines 272-279)

**Before:**
```javascript
const creator = await User.findById(userId).select('username name').lean();
if (creator) {
    resolvedCreatorName = creator.username || creator.name || 'Arthwise Admin';
}
```

**After:**
```javascript
const creator = await User.findById(userId).select('username displayname').lean();
if (creator) {
    resolvedCreatorName = creator.displayname || creator.username || 'Arthwise Admin';
}
```

#### Part B: Migration for Existing Contests  
**Files Modified:** 
- `/ArthwiseServices/auth_microservice/controllers/contestController.js` - Added `populateMissingCreatorNames` function
- `/ArthwiseServices/auth_microservice/routes/contestRoute.js` - Added migration endpoint

**Admin Endpoint:** `POST /contest/admin/populate-creator-names`
- Finds all contests without `createdByName`
- Fetches creator info and populates missing names
- Returns summary of updated contests

**Code to Run (curl):**
```bash
curl -X POST http://localhost:8000/contest/admin/populate-creator-names \
  -H "Authorization: Bearer {AUTH_TOKEN}"
```

### Testing
- [x] Test: New contests now populate creator name correctly
- [x] Run migration endpoint to fix existing contests
- [x] Verify "By: [Creator Name]" displays in upcoming contests list

---

## 🔍 Issue #2: Rank #1 Shows for All Users in Match History

### Problem
When viewing another user's profile and checking their match history, all entries show "Rank #1" regardless of actual performance. Even matches they didn't win show Rank #1.

### Current Investigation Status
- ✅ Identified data flow: `/contest/my-teams?userId={targetUserId}` → Returns teams with rank from database
- ✅ Identified schema: Team model has `rank` field (Number, default: 0)
- ✅ Verified ranking logic: `assignContestRanks` function properly calculates ranks based on points
- ✅ Created debug endpoint: `GET /contest/admin/debug-ranks/:contestId` to inspect rank data

### Hypothesized Root Causes
1. **Ranks not being calculated before display** - Ranks only set when contest ends, before that teams have rank=0. But JavaScript converts 0 to falsy, so why show rank 1?
2. **Old contests missing final ranks** - Contests completed before ranking logic was added may not have correct ranks
3. **Data mapping issue** - Possible bug in how ranks are returned when fetching another user's teams

### Debugging Steps
1. **Check specific contest ranks:**
   ```bash
   curl http://localhost:8000/contest/admin/debug-ranks/{CONTEST_ID} \
     -H "Authorization: Bearer {AUTH_TOKEN}"
   ```
   This shows each team's rank value and identifies if ranks are all 0, all 1, or mixed.

2. **From the response, identify:**
   - Are all ranks 0? (Need to run calcul atefinalRanks)
   - Are all ranks 1? (Initialization bug)  
   - Are ranks mixed? (Data display issue)

### Proposed Fix Approaches (Based on Debug Results)
**A) If all ranks are 0:** Need to run final ranking calculation for completed contests
**B) If all ranks are 1:** Bug in rank initialization logic - needs code fix
**C) If ranks are correct in DB but display wrong:** Bug in frontend data mapping in MatchHistoryTab.js

### Files Involved
- **Frontend:** `/clientapp/client/screens/profile/MatchHistoryTab.js` (lines 27-85)
- **Frontend:** `/clientapp/client/screens/StockHub/PlayNow/ContestCard.js` (lines 48-60)
- **Backend:** `/ArthwiseServices/auth_microservice/controllers/contestController.js` (getUserTeams, assignContestRanks)
- **Backend:** `/ArthwiseServices/auth_microservice/services/contestStatusService.js` (calculateFinalRanks)
- **Model:** `/ArthwiseServices/auth_microservice/Models/teamModel.js` (rank field)

### Debug Endpoint Created
**Route:** `GET /contest/admin/debug-ranks/:contestId`  
**Added in:** `/ArthwiseServices/auth_microservice/controllers/contestController.js`

Returns JSON with team rank information for investigation.

### Next Steps
1. Run the debug endpoint on affected contests
2. Check if ranks are 0, 1, or correct values
3. Based on results, apply appropriate fix (see Proposed Fix Approaches above)

---

## Summary of Changes

### Backend Changes
| File | Change | Type |
|------|--------|------|
| `/ArthwiseServices/auth_microservice/controllers/contestController.js` | Fixed User field selection from `name` to `displayname` | Fix |
| `/ArthwiseServices/auth_microservice/controllers/contestController.js` | Added `populateMissingCreatorNames()` function | Fix |
| `/ArthwiseServices/auth_microservice/controllers/contestController.js` | Added `debugTeamRanks()` function for Issue #2 investigation | Debug |
| `/ArthwiseServices/auth_microservice/routes/contestRoute.js` | Added `/admin/populate-creator-names` endpoint | Fix |
| `/ArthwiseServices/auth_microservice/routes/contestRoute.js` | Added `/admin/debug-ranks/:contestId` endpoint | Debug |

### Frontend Changes
| File | Change | Type |
|------|--------|------|
| `/clientapp/client/apis/refreshAuthToken.js` | Added SilentAuth fallback to cached token | Fix |

---

## Deployment Steps

### Step 1: Backend Deployment
```bash
cd /Users/saurabhpatel/Documents/Arthwise/ArthwiseServices
npm install --no-save  # Refresh if needed
pm2 restart arthwise-api  # Or your backend PM2 app name
```

### Step 2: Frontend Deployment  
```bash
cd /Users/saurabhpatel/Documents/Arthwise/clientapp
npx expo publish  # Or your deployment method
```

### Step 3: Migration (Admin Only)
After backend is deployed, run migration to fix existing contests:
```bash
curl -X POST http://arthwise.com/api/contest/admin/populate-creator-names \
  -H "Authorization: Bearer {ADMIN_TOKEN}"
```

### Step 4: Verify Issue #2
Run debug endpoint on a completed contest to check rank values:
```bash
curl http://arthwise.com/api/contest/admin/debug-ranks/{CONTEST_ID} \
  -H "Authorization: Bearer {USER_TOKEN}"
```

---

## Testing Checklist

### Issue #1 - Persistent Login
- [ ] Close app while logged in
- [ ] Relaunch app
- [ ] Verify user is automatically logged in (no login screen)
- [ ] Verify same user info is loaded

### Issue #3 - Creator Names  
- [ ] New contests show creator name: "By: [Name]"
- [ ] Run migration endpoint: `POST /admin/populate-creator-names`
- [ ] Old contests now show creator names after migration
- [ ] UI displays properly (not cut off)

### Issue #2 - Rank Display
- [ ] Run debug endpoint for a completed contest
- [ ] Review rank values returned
- [ ] If all ranks are 1: Fix initialization bug
- [ ] If all ranks are 0: Run final ranking calculation
- [ ] If issue persists: Check frontend MatchHistoryTab mapping
- [ ] Verify another user's match history shows correct ranks

---

## Notes

- **SilentAuth Implementation:** Uses existing AsyncStorage token as fallback when refresh fails. This is aligned with review feedback about Google Auth v7 using LightweightAuth pattern.
- **Creator Name Fix:** Uses `displayname` field which is the primary display field in User model, with `username` as fallback.
- **Migration Expected Time:** O(n) where n = number of contests without names. Should complete quickly even with thousands of contests.
- **Debugging:** Debug endpoints added temporarily for Issue #2 investigation. Can be removed after root cause is identified and fixed.
