# Schwaben Sentinels – Player Availability App
## Technical Specification & Feature Documentation
**Version:** 1.1  
**Last Updated:** March 2026  
**Created by:** iPrade  
**Club:** TSV Gr. Deinbach 1897 e.V.

---

## 1. Overview

A web-based player availability and team management application for **Schwaben Sentinels** cricket club. The app is hosted on **Netlify** (free tier) and uses **Firebase Realtime Database** as a shared backend so all players see the same data in real time.

**Live URL:** schwaben-sentinels.netlify.app  
**Technology Stack:**
- Frontend: Single HTML file (HTML + CSS + JavaScript)
- Database: Firebase Realtime Database (Google)
- Hosting: Netlify (free plan)
- Excel Export: SheetJS (xlsx.full.min.js)
- PDF Export: jsPDF + jsPDF-AutoTable

---

## 2. Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | Mar 2026 | Initial release — match availability, practice, overview, activity, admin, reminders |
| v1.1 | Mar 2026 | Added Team tab, Photos tab, Match Summary, Achievements, Improvement tags, photo permissions |

---

## 3. Architecture

```
Player's Browser
     ↓ opens URL
Netlify (serves HTML file)
     ↓ browser loads HTML
Firebase Realtime Database (stores all shared data)
```

All data is stored in Firebase under the `tsvgd` key. The app falls back to `localStorage` if Firebase is unavailable (offline mode).

### Firebase Data Structure
```
tsvgd/
├── players/              # Player availability data
│   └── {playerName}/
│       └── {league}_{matchIdx}/
│           ├── status        # yes | maybe | no
│           ├── reason        # text
│           ├── notes         # text
│           └── lastEdit      # ISO timestamp
├── pins/                 # Hashed player PINs
│   └── {playerName}: hash
├── profiles/             # Player profile information
│   └── {playerName}/
│       ├── jersey, role, batting, bowling
│       ├── dob, joined, contact, emergency
│       ├── specialRoles: [Captain, Vice Captain, etc.]
│       └── photoURL: base64 data URL
├── improvements/         # Improvement tags (admin + player only)
│   └── {playerName}: [improvementId, ...]
├── achievements/         # Match achievements per player
│   └── {league}_{matchIdx}/
│       └── {playerName}/
│           └── badges: [achievementId, ...]
├── matchSummaries/       # Match result summaries
│   └── {league}_{matchIdx}/
│       ├── result, ourScore, oppScore
│       ├── motm, bomm, notes
│       ├── savedBy, savedAt
├── practice/             # Practice sessions
│   └── {sessionId}/
│       ├── id, date, time, type, location, mapLink, notes
│       ├── createdBy
│       ├── responses/
│       │   └── {playerName}: {status, reason, notes, lastEdit}
│       └── attendance/
│           └── {playerName}: present | absent
├── friendly/             # Friendly matches
│   └── {matchId}/
│       ├── id, opponent, date, time, format, homeAway, location, notes
│       ├── createdBy
│       └── responses/
│           └── {playerName}: {status, reason, notes, lastEdit}
├── photos/               # Photo albums (base64 stored in DB)
│   └── {albumKey}/       # team | match_{key} | practice_{id}
│       └── {photoId}/
│           ├── id, dataURL, caption
│           ├── uploadedBy, uploadedAt
│           ├── fileName, fileSize
├── grounds/              # Practice grounds list
│   └── [{name, address, mapLink}]
├── permissions/          # Access control
│   ├── admins: [playerName, ...]
│   ├── canCreateSessions: [playerName, ...]
│   └── canUploadPhotos: [playerName, ...]
├── settings/             # App settings
│   ├── warnThreshold     # Late cancellation warning count
│   ├── defaultGround     # Default ground name
│   └── defaultGroundIdx  # Index in grounds array
├── lateCancellations/    # Late change tracking
│   └── {playerName}: [{label, time}]
├── adminUnlocks/         # Admin match unlock overrides
│   └── {league}_{idx}_{player}: true
└── lastEdits/            # Last edit timestamps
    └── {playerName}: ISO timestamp
```

---

## 4. Match Data

### DCB Verbandsliga Südwest 2026 – 50 Overs Group B
9 matches for TSV Großdeinbach (MD1: 26 Apr → MD9: 06 Sep 2026)

| Match Day | Date | Home | Away | Venue |
|-----------|------|------|------|-------|
| MD 1 | 26.04.2026 | Vfb Friedrichshafen | TSV Großdeinbach | Away |
| MD 2 | 03.05.2026 | TSV Großdeinbach | Aalener SA | Home |
| MD 3 | 10.05.2026 | TSV Großdeinbach | SSV Ulm | Home |
| MD 4 | 17.05.2026 | TSV Großdeinbach | SVC Knights | Home |
| MD 5 | 24.05.2026 | TSV Großdeinbach | Spvg Feuerbach | Home |
| MD 6 | 31.05.2026 | TSV Großdeinbach | MTV BP 2 | Home |
| MD 7 | 07.06.2026 | FVP Spades | TSV Großdeinbach | Away (FV Plochingen 2) |
| MD 8 | 30.08.2026 | TSV Großdeinbach | SVP Blitz | Home |
| MD 9 | 06.09.2026 | TSV Großdeinbach | MTV BP U21 | Home |

### DCB T20 Verbandsliga Südwest 2026 – Group A
11 matches for TSV Großdeinbach (MD1: 21 Jun → MD9: 20 Sep 2026)

| Match Day | Date | Home | Away | Venue |
|-----------|------|------|------|-------|
| MD 1 | 21.06.2026 | TSV Großdeinbach | MTV BP 2 | Home (11:00) |
| MD 1 | 21.06.2026 | TSV Großdeinbach | MTV BP U21 | Home (15:00) |
| MD 2 | 28.06.2026 | Spvg Feuerbach | TSV Großdeinbach | Away |
| MD 3 | 05.07.2026 | HNCC 2 | TSV Großdeinbach | Away (15:00) |
| MD 4 | 11.07.2026 | FVP Spades | TSV Großdeinbach | Away (11:00) |
| MD 4 | 11.07.2026 | FVP Aces | TSV Großdeinbach | Away (15:00) |
| MD 5 | 19.07.2026 | TSV Großdeinbach | TSVM Hawks | Home (11:00) |
| MD 5 | 19.07.2026 | TSV Großdeinbach | SVP Blitz | Home (15:00) |
| MD 6 | 26.07.2026 | SSV Ulm | TSV Großdeinbach | Away |
| MD 7 | 16.08.2026 | Vfb Friedrichshafen 2 | TSV Großdeinbach | Away |
| MD 9 | 20.09.2026 | TSV Großdeinbach | SVC Knights | Home |

---

## 5. Features

### 5.1 Authentication — PIN System
- Each player logs in with their **name + 4-digit PIN**
- First login: player sets their own PIN (confirmed twice)
- Returning login: player enters name → enters PIN
- PIN is **hashed** before storage (not stored in plain text)
- Default admin PIN: **1897** (club founding year)
- Admin PIN stored in `localStorage` (device only, not shared)
- Players can only edit **their own** availability data

### 5.2 Navigation
- **Desktop:** Tab bar across the top
- **Mobile (portrait):** Dropdown menu (tap to expand)
- Tabs: 50 Overs | T20 | Friendly | Practice | Overview | Activity | **Team** | **Photos** | Admin

### 5.3 Match Availability (50 Overs & T20 tabs)
- Per match card: Match Day, Date, Time, Home vs Away, Venue
- Player selects: **✓ Available / ? Maybe / ✗ Unavailable**
- If Maybe or Unavailable: reason + notes fields appear
- Team Overview shows all teammates' responses per match
- **Match Summary section** (v1.1) — shows result, scores, MOTM, BOMM, achievements

#### 16-Day Lock Rule
- Players **cannot change** status within 16 days of a match date
- Locked buttons are greyed out with a lock message
- Admin can **unlock a specific match for a specific player**
- Any change in locked period is flagged as a **late change**

### 5.4 Match Summary (v1.1)
Added by admin or permitted players after each match:
- **Result:** Win / Loss / Draw / No Result
- **Scores:** Our score vs opponent score
- **Man of the Match** — dropdown of registered players
- **Bowler of the Match** — dropdown of registered players
- **Match notes** — free text
- **Player achievements** — checkboxes per player per match
- Visible to all logged-in players inside the match card

### 5.5 Friendly Matches Tab
- Admin or permitted players create friendly matches
- Fields: Opponent, Date, Time, Format, Home/Away, Location, Notes
- Players respond: Available / Maybe / Unavailable + reason + notes
- Team overview shown per match

### 5.6 Practice Tab
Two views toggled at top:

#### Sessions View
- Monthly navigation (◀ Month ▶)
- Stats: Sessions count, Avg Committed, Response Rate
- Pending summary chips
- Per session: Type badge, Date, Time, Ground (clickable map link), Notes
- Response: Available / Maybe / Unavailable + reason + notes
- **24-hour lock rule**
- Past sessions show Attendance Record

#### Attendance View
- Summary cards: Past Sessions, Avg Attendance %, Avg Committed %, Lost
- Full grid: Players × Sessions
- Cell codes: ✓ Present, ✗ Absent, ! Committed/Absent, – Not marked
- Per-player totals: Attendance %, Committed %, Lost count

#### Practice Grounds (admin managed)
- Saved grounds: Name + Address + Google Maps link
- Dropdown when creating sessions + custom text option
- Default ground configurable in Settings

### 5.7 Overview Tab
Three views: **50 Overs | T20 | Practice**

#### Match Overview
- **First pinned row:** "Confirmed ✓" — total confirmed + % + maybe count per match
- Player rows: colour-coded status cells
- Footer row: available count + percentage
- Export: Excel (2 sheets) + PDF (landscape A4)

#### Practice Overview
- Same attendance grid as Practice → Attendance view

### 5.8 Activity Tab
- Players sorted by most recently edited
- Per player: avatar, last edit time, response summary
- Per match: date, opponent, status, exact timestamp
- **Late change flag:** 🔴 if status changed within 16 days
- Toggle: 50 Overs | T20 | All

### 5.9 Team Tab (v1.1)
Card grid with expandable details for each registered player.

#### Player Card (collapsed)
- Circular photo thumbnail (or initials avatar)
- Full name, role, jersey number
- Special role badges: 👑 Captain, ⭐ Vice Captain, 🧤 Wicket Keeper, ⚖️ Umpire, 🎓 Coach
- Batting style, bowling style
- Achievement count + recent badges

#### Player Card (expanded)
- DOB, joined year
- Contact number, emergency contact
- Edit Profile button (own profile or admin)
- Upload/update photo button (own profile)

#### Filters & Sorting
- Filter by role (Batsman/Bowler/All-rounder/WK)
- Sort by name / jersey number / achievements

#### Edit Profile Modal
- Jersey number, role, batting/bowling style
- DOB, joined year, contact, emergency contact
- Special roles checkboxes
- Improvement areas (admin only section, shown to admin + player)

### 5.10 Achievements System (v1.1)

#### Achievement Badges (visible to ALL players)
Awarded per match by admin or permitted players:

| Badge | ID | Emoji |
|-------|-----|-------|
| Man of the Match | motm | 🏆 |
| Best Batsman | bestbat | 👏 |
| Best Bowler | bestbowl | 🎯 |
| Best Fielder | bestfield | 🧤 |
| Half Century 50+ | fifty | 🔶 |
| Century 100+ | century | 🔶🔶 |
| 5-Wicket Haul | fifer | 🔥 |
| Hat-trick | hattrick | 🎓 |
| Best Catch | bestcatch | 🧤 |
| Run Out | runout | ⚡ |
| Most Improved | improved | 📈 |
| Best Partnership | partnership | 🤝 |
| Super Over Hero | superover | ⭐ |
| Comeback Player | comeback | 💪 |
| Highest Run Rate | runrate | ⚡ |
| Most Dot Balls | dotballs | 🔴 |
| Economy King | economy | 👑 |
| Highest Score | highscore | 🏅 |
| Best Opening Partnership | opening | 🏆 |
| Best Death Bowling | deathbowl | 💣 |
| Custom | custom | 🏷 |

Displayed on: match card summary + player profile card

#### Improvement Tags (visible to admin + player only)
| Tag | ID | Emoji |
|-----|-----|-------|
| Work on fitness | fitness | 🏃 |
| Improve fielding | fielding | 🧤 |
| Focus on batting technique | batting | 🥊 |
| Work on line & length | linelength | 🎯 |
| Improve running between wickets | running | 🏃 |
| Improve catching | catching | 🧤 |
| Communication on field | comms | 🗣 |
| Work on death bowling | deathbowl | 💣 |
| Improve footwork | footwork | 👣 |
| Stay focused | focus | 🧠 |
| Needs more practice | practice | 🏋 |

### 5.11 Photos Tab (v1.1)
Three albums: **Match | Practice | Team**

#### Features
- List view: photo thumbnail (180px) + caption + uploader + timestamp
- Storage usage bar (out of 5GB Firebase free tier)
- Max 5MB per photo
- Supported formats: JPG, PNG, GIF, HEIC/HEIF (iPhone), WebP
- iPhone tip: Settings → Camera → Formats → Most Compatible
- Photos stored as base64 data URLs in Firebase

#### Permissions
- Admin selects which players can upload (+ Photos permission in Admin panel)
- Any player can view all photos
- Uploader can delete their own photos
- Admin can delete any photo

#### Albums
- **Match:** Select match from dropdown → upload/view photos for that match
- **Practice:** Select session from dropdown → upload/view photos
- **Team:** General club album

### 5.12 Admin Panel
Protected by admin PIN (default: **1897**).

#### Player Management
| Action | Description |
|--------|-------------|
| Rename | Rename player, all data migrates |
| Reset PIN | Clears PIN, player sets new one |
| Clear Data | Wipes availability, keeps account |
| Delete | Removes completely |
| + Creator | Grants session creation permission |
| + Photos | Grants photo upload permission |
| Promote/Demote | Admin role toggle |

#### Settings
- Late cancellation warning threshold (default: 3)
- Default practice ground (dropdown from saved list)

#### Practice Grounds
- Add / Edit / Delete grounds
- Name + Address + Google Maps link

#### Match Unlock
- Unlock specific match for specific player within 16-day lock

### 5.13 Reminder & Progress System
| Feature | Location | Covers |
|---------|----------|--------|
| Progress bar | Below player banner | Matches + practice |
| Tab badges | Tab labels | Pending count per tab |
| Login reminder | Top of content | Most urgent pending items |
| Pending summary | Top of each tab | Chips linking to each match |

### 5.14 Late Cancellation Tracking
- Matches: 16-day lock
- Practice: 24-hour lock
- Counter in Admin panel per player
- 🔴 flag in Activity log
- Warning at admin-configurable threshold
- Included in exports

### 5.15 Permissions Summary
| Role | Capabilities |
|------|-------------|
| Regular player | View all, edit own availability + own profile |
| Session Creator | Above + create/edit practice + friendly sessions |
| Photo Uploader | Above + upload photos to any album |
| Admin | Full access — all above + player management + settings + unlocks |

---

## 6. Branding
- **Team name:** Schwaben Sentinels
- **Club:** TSV Gr. Deinbach 1897 e.V.
- **Logo:** Embedded base64 PNG (no external dependency)
- **Colours:** Dark green (#0b1610), green (#1a6b3c), gold (#c8a84b)
- **Fonts:** Bebas Neue (headings), Outfit (body), JetBrains Mono (dates/codes)
- **Footer:** "Created by iPrade · Schwaben Sentinels · v1.1"

---

## 7. Deployment

### Netlify
- **URL:** schwaben-sentinels.netlify.app
- **Plan:** Free (Spark)
- **Method:** Netlify Drop or drag to deploy box in dashboard
- **Update process:** Log in → site dashboard → drag new HTML file

### Firebase
- **Project:** schwaben-sentinels
- **Database URL:** schwaben-sentinels-default-rtdb.europe-west1.firebasedatabase.app
- **Region:** europe-west1 (Belgium)
- **Plan:** Spark (free) — 1GB DB, 5GB Storage, 10GB/month bandwidth
- **Rules:** Public read/write (app-level PIN protection)

### Migration Plan (Personal → Team Account)
1. Create team Gmail: schwaben.sentinels@gmail.com
2. Create new Firebase project with team Gmail
3. Export data from old Firebase, import to new
4. Update Firebase config in HTML
5. Create GitHub account with team Gmail
6. Connect GitHub to Netlify for auto-deploy
7. Connect GitHub to Claude for version control

---

## 8. Planned / Future Features

### Next (v1.2)
- **GitHub integration** — auto-deploy on commit, full version history
- **Team account migration** — move from personal to club Gmail/Firebase

### Backlog
- **Scorecard tab** — publish match scorecards (runs, wickets, overs, player scores)
- **Scorecard per friendly** — same for friendly matches
- **Push notifications** — remind players to update availability

---

## 9. File Reference

| File | Description |
|------|-------------|
| `schwaben-sentinels-v1.1.html` | Current application (v1.1, ~310KB) |
| `schwaben-sentinels-v1.0.html` | Previous version backup |
| `schwaben-sentinels-spec.md` | This document |
| Firebase project | schwaben-sentinels (console.firebase.google.com) |
| Netlify project | schwaben-sentinels (app.netlify.com) |

---

## 10. Admin Quick Reference

| Task | How |
|------|-----|
| Log in as admin | Admin tab → enter PIN 1897 |
| Change admin PIN | Admin → Change PIN |
| Add match summary | Match card → + Add Match Summary |
| Award achievements | Match card → Edit Summary → Player Achievements |
| Edit player profile | Team tab → player card → expand → Edit Profile |
| Set player role (Captain etc.) | Edit Profile → Special Roles |
| Add improvement tags | Edit Profile → Improvement Areas (admin only) |
| Upload player photo | Team tab → player card → expand → Update photo |
| Grant photo upload permission | Admin → player row → + Photos |
| Add practice ground | Admin → Practice Grounds → + Add Ground |
| Set default ground | Admin → Settings → Default Ground |
| Export match data | Overview tab → Excel / PDF |
| Export practice data | Practice tab → Excel / PDF |
| Unlock match for player | Admin → adminUnlockMatch(league, idx, player) |

---

*Document version: 1.1 | Generated: March 2026 | Created by iPrade*

---

## 11. v1.2 Changes

### New Features

**🏏 Matches Tab**
- Separate tab for all match-related features
- Sub-tabs: 50 Overs | T20 | Friendly | Standings
- Upcoming matches at top, past matches below per sub-tab
- Per match card: result badge, scores, MOTM, BOMM, achievements, notes, scoresheet PDF, head-to-head
- Season standings table: P/W/L/D/NR/Points (auto-calculated)
- Head to head record vs each opponent on every card

**📅 Dynamic Match Schedule**
- Matches no longer hardcoded — stored in Firebase under `schedule/{league}_{season}/{matchId}`
- 2026 matches auto-migrated from hardcoded data on first load (`initScheduleFromHardcoded`)
- Admin/Team Manager can add, edit, delete matches
- Fields: Match Day, Date, Time, League, Home team, Away team, Venue, Season
- Season management: Admin creates seasons, sets active season in Settings
- Old seasons archived but preserved

**🎖️ Role System (replaces Creator/Photos/Promote)**

| Role | Badge | Permissions |
|------|-------|------------|
| Team Manager | 📋 | schedule, sessions, friendly |
| Scorer | 📊 | results, scoresheet |
| Photographer | 📸 | photos |
| Fitness Coach | 🏋 | practice, attendance |
| Announcer | 📢 | notes |
| Selector | 🎯 | achievements |
| Batting Coach | 🏏 | practice |
| Bowling Coach | 🎳 | practice |
| Fielding Coach | 🧤 | practice |
| Social Media Mgr | 📱 | badge only |
| Equipment Manager | 🪄 | badge only |
| + Custom roles | any emoji | admin defines perms |

- Admin assigns role → player gets Accept/Decline prompt on next login
- Accepted role badges shown on Team tab profile cards
- Role badges shown alongside special role badges (Captain, WK etc.)

**🏆 Custom Achievements & Improvement Tags**
- Admin creates custom achievement badges (name + emoji)
- Admin creates custom improvement tags (name + emoji)
- Appear alongside built-ins in all achievement selectors

**⚙️ Updated Settings**
- Team name (used in match schedule and standings)
- Active season selector

### Firebase Data Structure Additions (v1.2)
```
tsvgd/
├── schedule/             # Dynamic match schedule
│   └── {league}_{season}/
│       └── {matchId}/
│           ├── id, day, dateISO, date, time
│           ├── league, season, home, away, venue, location
├── playerRoles/          # Responsibility role assignments
│   └── {playerName}/
│       └── {roleId}: pending | accepted | declined
├── customRoles/          # Admin-created custom roles
│   └── [{id, emoji, label, perms}]
├── customAchievements/   # Admin-created custom achievement badges
│   └── [{id, emoji, label}]
├── customImprovements/   # Admin-created custom improvement tags
│   └── [{id, emoji, label}]
├── seasons/              # List of all seasons
│   └── ['2026', '2027', ...]
├── scheduleInitialised/  # Flag to prevent re-migration of hardcoded data
```

*Document version: 1.2 | Generated: March 2026 | Created by iPrade*

---

## 12. v1.3 Changes — Enhanced Security Model

### Overview
v1.3 replaces the open name-entry login with a fully controlled access system. Only admin-registered players can log in. Players choose between PIN or password authentication.

### New Login Flow

**Before (v1.2):**
- Anyone could type any name and join
- Single auth method (4-digit PIN)

**After (v1.3):**
- Dropdown only — no free text entry
- Only admin-approved names visible
- First login requires temp PIN from admin
- Player forced to set own PIN or password on first login

### Detailed Flow

#### First Login
1. Player selects name from dropdown
2. Enters temporary PIN provided by admin
3. Redirected to **Choose Security** screen
4. Chooses: **4-digit PIN** or **Password (min 6 chars)**
5. Sets and confirms their chosen method
6. Logged in ✅

#### Returning Login
1. Select name from dropdown
2. PIN users → keypad screen (same as before)
3. Password users → text input screen with keyboard support
4. Logged in ✅

#### Temp PIN
- Admin sets a temporary PIN when adding a player
- Player shown as **"Temp PIN"** badge in Admin panel
- On first login with temp PIN → forced to choose own security
- Badge removed once player sets their own PIN/password

### Admin — Add Player
Location: Admin panel → Registered Players → **+ Add Player**

Fields:
- **Full Name** — player's name (appears in dropdown)
- **Temporary PIN** — 4-digit number, share with player verbally or via message
- **Email** (optional) — stored in profile for future password reset feature

### Authentication Options

| Method | Format | When chosen |
|--------|--------|-------------|
| 4-digit PIN | Numbers only, keypad UI | Quick access, simple |
| Password | Min 6 chars, text input | More secure |

- Both stored as hashed values (never plain text)
- Player can switch between PIN and password anytime
- Change security: Team tab → own profile card → expand → 🔐 Change PIN/Password

### Firebase Data Structure Additions (v1.3)
```
tsvgd/
├── authType/             # Auth method per player
│   └── {playerName}: pin | password
├── tempPins/             # Tracks which players still have admin temp PIN
│   └── {playerName}: true | (deleted when player sets own)
├── profiles/             # Email added
│   └── {playerName}/
│       └── email: string (optional)
```

### Backward Compatibility
- All existing players and their PINs are preserved
- Existing players are NOT forced to re-register
- They log in normally with existing PIN
- They can optionally switch to password via their profile

### Security Notes
- All PINs and passwords hashed before storage (same hash function)
- Admin PIN still stored in localStorage (device only)
- Player credentials stored in Firebase under `pins/{playerName}`
- No plain text credentials stored anywhere
- Email stored for future password reset (not yet implemented)

---

## 13. Version Summary

| Version | Date | Key Changes |
|---------|------|-------------|
| v1.0 | Mar 2026 | Initial — availability, practice, overview, activity, admin, reminders |
| v1.1 | Mar 2026 | Team tab, Photos tab, Match Summary, Achievements, Improvement tags |
| v1.2 | Mar 2026 | Matches tab, Dynamic schedule, Role system, Custom badges, Season management |
| v1.3 | Mar 2026 | Controlled login (dropdown only), Admin adds players, PIN + Password auth |

---

## 14. Pending / Future Features

| Feature | Priority | Notes |
|---------|----------|-------|
| Password reset via email | Medium | Email field already collected in v1.3 |
| GitHub auto-deploy | Medium | Connect repo to Netlify for CI/CD |
| Team account migration | Low | Move from personal to club Gmail/Firebase |
| Scorecard tab | Low | Batting/bowling figures per match |
| Push notifications | Low | Remind players to update availability |

---

*Document version: 1.3 | Generated: March 2026 | Created by iPrade*

---

## 15. v1.3.1 Patch — Backup & Restore + Login Fix

### Changes

**🔐 Login Fix (chicken-and-egg)**
- Problem: Fresh install had empty dropdown → nobody could log in
- Fix: Dropdown always shows **"⚙️ Admin Access"** at the bottom
- Selecting Admin Access → app opens → Admin tab auto-launches
- Admin can then add players via + Add Player
- Hint shown on login: *"First time? Select ⚙️ Admin Access at the bottom of the list."*
- When no players exist yet, subtitle reads: *"No players registered yet. Use Admin Access to set up the app."*

**💾 Backup & Restore**
Location: Admin panel → Backup & Restore section (top of admin dashboard)

| Feature | Description |
|---------|-------------|
| ⬇ Export Backup | Downloads all data as timestamped JSON file |
| ⬆ Import Backup | Restores data from a JSON backup file |
| Last backup date | Shows when last backup was taken, colour coded |

**Export format:**
```json
{
  "_meta": {
    "exportedAt": "2026-03-16T10:30:00.000Z",
    "exportedBy": "Pradeep",
    "appVersion": "v1.3",
    "club": "Schwaben Sentinels"
  },
  "data": { ...all Firebase data... }
}
```

**Last backup indicator colours:**
- 🟢 Green — backed up today or this week
- 🟡 Amber — backed up 7–30 days ago
- 🔴 Red — never backed up or over 30 days ago

**Backup storage:** Downloaded to local device (not stored in Firebase). Recommend saving to Google Drive or cloud storage.

**Recommended backup schedule:**
- Before every Netlify deploy
- Start and end of each season
- After major data entry (match results, achievements)

**Last backup date** stored in `localStorage` (device only — not shared across devices).


---

## 16. v1.4 — Scorecard System

**Date:** March 2026

### New Features

**🏏 Scorecard per Match**
- Shown inside each match card in the Matches tab
- Both innings: Our innings + Opponent innings
- Batting: Name, Runs, Balls, 4s, 6s, How Out, Bowler, Strike Rate (auto)
- Bowling: Name, Overs, Maidens, Runs, Wickets, Economy (auto), Wides, No Balls
- Extras per innings: Wides, No Balls, Byes, Leg Byes
- Total, Wickets, Overs per innings

**Auto-highlights shown above scorecard:**
- 🏏 Top Scorer — highest runs + score
- 🎯 Top Bowler — most wickets
- ⭐ Half Century badge (50+)
- 💯 Century badge (100+)
- 🔥 5-Wicket Haul badge

**Who can enter:** Admin + Scorer role

**🤖 Method 1 — AI Import (Claude)**
- Upload photo (JPG/PNG/HEIC) or PDF of any scoresheet
- Claude reads handwritten, printed, any format
- Auto-populates all batting + bowling figures
- Scorer reviews and saves
- Works with photos taken on iPhone

**📊 Method 2 — Excel Template**
- Download pre-formatted template (4 sheets: Our Batting, Our Bowling, Opp Batting, Opp Bowling)
- Fill in offline
- Upload back — auto-populates scorecard
- Falls back to manual review/editing

**Manual entry always available** — enter directly via form

**⬇ PDF Export**
- Full scorecard exported as landscape A4 PDF
- Both innings, dark green theme matching app

**👤 Career Stats on Player Profiles**
Shown in player card expanded section:

| Stat | Description |
|------|-------------|
| Matches | Total matches with scorecard data |
| Total Runs | Career run tally |
| High Score | Personal best |
| Batting Avg | Runs / (Innings - Not Outs) |
| Strike Rate | (Runs / Balls) × 100 |
| 100s | Centuries |
| 50s | Half centuries |
| Wickets | Career wicket tally |
| Best Figures | Best bowling e.g. 5/32 |
| Economy | Runs per over |

### Firebase Data Structure Additions (v1.4)
```
tsvgd/
├── scorecards/           # Match scorecards
│   └── {matchId}/
│       ├── ourInnings/
│       │   ├── batting: [{name,runs,balls,fours,sixes,howOut,bowler}]
│       │   ├── bowling: [{name,overs,maidens,runs,wkts,wides,noballs}]
│       │   ├── extras: {wd,nb,b,lb}
│       │   ├── total, wkts, overs
│       ├── oppInnings/   (same structure)
│       ├── savedBy, savedAt
│       ├── importedViaAI: true (if AI import used)
│       └── importedViaExcel: true (if Excel import used)
```


---

## 17. v1.5.0 — UI Improvements & Bug Fixes

**Date:** March 2026

### Matches Tab — Scorecard as Tab
- Scorecard no longer shown expanded below every match card
- Each match card now has **📋 Details** and **🏏 Scorecard** toggle buttons in the header
- Clicking a button expands that section inline; clicking again collapses it
- Both sections are collapsed by default — clean, uncluttered cards

### Matches Tab — Venue Address + Google Maps
- Venue block added below team names in each match card
- Shows venue name + optional address line
- **🗺 Maps** button links directly to Google Maps search for the venue
- Address field added to the Add/Edit Match form

### Availability Tabs — Cleaned Up
- Removed scorecard section from availability match cards (moved to Matches tab only)
- Removed team overview pills from availability cards
- Cards now show: match info + **Your Availability** buttons only — clean and focused

### Activity Log — Fixed
- Fixed silent crash caused by `canUploadPhotos` reference (removed in v1.3)
- Activity log now correctly shows all players and their match responses

### Team Tab — Fixed
- Fixed silent crash caused by `ACHIEVEMENTS`, `IMPROVEMENTS`, `SPECIAL_ROLES` direct references (renamed in v1.2)
- All player cards now render correctly with achievement badges and improvement tags

### Special Roles — Admin Only
- Captain, Vice Captain, WK, Umpire, Coach can only be assigned by admin in Edit Profile
- Players see their current roles as read-only badges with a note
- **New roles:** 50 Overs Captain, 50 Overs Vice Captain, T20 Captain, T20 Vice Captain (separate per format)

### Reminder Banner — Scroll Added
- Login reminder banner capped at 300px with internal scroll
- Pending chips list capped at 120px with scroll
- ✕ dismiss button added to banner title
- Thin green scrollbar styling

### Role Assignment — Immediate
- Roles now auto-accepted on assignment — badge appears instantly on Team tab
- Player receives acknowledgement notification on next login (OK / Remove)

