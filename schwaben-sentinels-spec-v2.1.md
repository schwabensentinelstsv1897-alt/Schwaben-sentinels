# Schwaben Sentinels – Player Availability App
## Technical Specification & Feature Documentation
**Version:** 2.1  
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
| v1.6.2 | Mar 2026 | Critical Firebase sync fix — `window.__fbSet` and `window.__fbGet` were missing |
| v1.7.0 | Mar 2026 | Photo gallery redesign — grid view, lightbox, drag & drop upload modal |
| v1.8.1 | Mar 2026 | Strip base64 dataURLs from Firebase writes — fixes 1MB node limit silent rejection |
| v1.8.3 | Mar 2026 | Granular `fbWrite(path, value)` — targeted writes replace monolithic `saveData()` |
| v1.8.6 | Mar 2026 | Firebase root key changed `tsvgd` → `ss2026` to avoid tombstone issue |
| v1.8.7 | Mar 2026 | Player nodes use `{_p:true}` instead of `{}` — Firebase prunes empty objects |
| v1.8.8 | Mar 2026 | PIN reset auto-generates temp PIN; `loginSuccess()` waits for Firebase writes |
| v1.8.9 | Mar 2026 | Mobile bottom tab bar replaces dropdown; auto temp PIN shown in admin modal |
| v1.9.0–v1.9.2 | Mar 2026 | Mobile tab bar live, photos coming soon placeholder, pending chip fixes |
| v1.9.3 | Mar 2026 | Bug Report tab + Admin consolidated view |
| v1.9.4 | Mar 2026 | Temp PIN stored and displayed in admin panel until player sets own PIN |
| v1.9.5–v1.9.8 | Mar 2026 | Match addresses fixed, upcoming banner redesigned, mass CSS restoration |
| v2.0.0 | Mar 2026 | Complete CSS restoration, all form layouts fixed, Home🟢/Away🔴, team/match bugs fixed |
| v2.0.1 | Mar 2026 | Bug report JS functions restored after v2.0.0 rebuild drop |
| v2.0.2 | Mar 2026 | Settings form now loads all 6 fields correctly |
| v2.0.3 | Mar 2026 | Help tab added in-app + PDF player guide with logo/header |
| v2.0.4 | Mar 2026 | Share bar and default PIN hint removed |
| v2.0.5 | Mar 2026 | More drawer closes on mobile tap, friendly badge, grouped pending reminder, help logo |

---

## 3. Architecture

```
Player's Browser
     ↓ opens URL
Netlify (serves HTML file)
     ↓ browser loads HTML
Firebase Realtime Database (stores all shared data)
```

All data is stored in Firebase under the `ss2026` key (changed from `tsvgd` in v1.8.6). The app falls back to `localStorage` (`ss2026-data`) if Firebase is unavailable (offline mode). The app falls back to `localStorage` if Firebase is unavailable (offline mode).

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


---

## 18. v1.6.0 — UI Polish, Maps, Activity Feed & Admin Improvements

**Date:** March 2026

### Matches Tab — Scorecard Access
- **🏏 Scorecard** tab button only appears on a match card if scorecard data has been entered
- No scorecard entered → only **📋 Details** shown, card stays clean
- Scorecard entered → green **🏏 Scorecard** button appears next to Details
- Scorecard always collapsed by default — opens on click only

### Matches Tab — Inline Score Summary
- When scorecard data exists, a compact result bar shows immediately on the match card (no click needed)
- Shows: Result badge (Win/Loss/Draw) + Our score + Opponent score + Man of the Match
- Styled in dark bar below team names

### Venue Addresses + Google Maps
All 2026 fixtures updated with full addresses from BWCV Clubs Ground Address document:

| Venue | Address |
|-------|---------|
| TSV Großdeinbach (Home) | Albert-Schweizer-Straße 17, 73527 Schwäbisch Gmünd |
| Vfb Friedrichshafen | Teuringer Str. 2, 88045 Friedrichshafen |
| Aalener SA | Hirschbachstadion, Hirschbachstraße 70, 73431 Aalen |
| SSV Ulm | Stadion Str. 17, 89073 Ulm |
| SVC Knights | Sportpark Weil, Weilstraße 199, 73733 Esslingen |
| Spvg Feuerbach | Am Sportpark 1, 70469 Stuttgart |
| FV Plochingen | Pfostenbergweg, 73207 Plochingen |
| HNCC 2 | Lange Wiesen 1, 74081 Heilbronn |
| MTV BP / SVP Blitz / TSVM Hawks / SVC Knights | TBC |

- **🗺 Maps** button links directly to Google Maps for each venue
- Address shown as text below venue name in both Matches tab and Availability cards
- TBC venues show "Address TBC" in grey italics

### Activity Log Redesign
- Replaced per-player expanded cards with a **compact feed** (last 15 edits, newest first)
- Each row: colour-coded avatar • player name • match • status • time ago
- Each player has a unique accent colour on the left border
- **Click any row** to expand full details (match date, reason, timestamp)
- **Late change** 🔴 flag shown inline
- **"Show all X updates"** button at bottom if more than 15
- `hexToRgb` helper added for dynamic row colouring

### Admin Panel — Make Admin / Demote
- Each player row now has **👑 Make Admin** button (gold)
- If player is already admin → shows **👑 Demote** button (red)
- Admin players show **👑 Admin** gold badge next to their name
- Player meta row now shows auth type (PIN set / Password set)

### Login Screen
- **"Who are you?"** replaced with **"Welcome, Sentinel!"**
- 🏏 bat emoji replaced with **Schwaben Sentinels club logo**
- Logo displayed as circular image with gold border

### Instagram Link
- Club Instagram link added to app header, next to 2026 season tag
- Shows Instagram icon + `@schwabensentinels`
- Configurable from Admin panel → Settings → **Instagram Handle** field
- Saves to Firebase — updates for all players instantly

### Special Roles — Format Specific Captains
- **50 Overs Captain** and **50 Overs Vice Captain** (separate from T20)
- **T20 Captain** and **T20 Vice Captain**
- Wicket Keeper, Umpire, Coach remain unchanged
- Special roles remain admin-only in Edit Profile

### Pending Build List (v1.7.0)
- 🖼️ Photo Gallery redesign (tiles + slideshow)
- 📊 Player stats: match-by-match breakdown, batting position, fielding stats (catches/stumpings/run outs/drop catches/misfields)
- 🏆 Season leaderboard (Most Runs, Wickets, Catches, Average, Economy)
- 🎨 Colour themes (admin selectable, 5 built-in + custom hex)


---

## 19. v1.6.1 — Colour Theme System

**Date:** March 2026

### Colour Themes
Admin can change the app theme from **Admin panel → Settings → Colour Theme**.
Theme saved in Firebase — applies instantly to all players.

**Available themes:**

| Theme | Emoji | Primary | Accent | Notes |
|-------|-------|---------|--------|-------|
| Forest Green | 🌲 | Deep green | Gold | Default |
| Sky Blue | 🔵 | Navy | Sky blue | Matches jersey colour |
| Midnight Black | ⚫ | Near black | Silver/grey | Winter |
| Crimson | 🔴 | Dark red | Gold | Away season |
| Royal Purple | 🟣 | Dark purple | Gold | Special events |

**How it works:**
- Admin selects theme → live preview immediately in the browser
- Click **💾 Save Settings** → saved to Firebase
- All connected players see the new theme on next data refresh
- Theme also applied from localStorage on page load (instant, no flash)
- Uses CSS `data-theme` attribute on `<html>` root element with CSS variable overrides


---

## 20. v1.6.2 — Critical Firebase Sync Fix

**Date:** March 2026

### Bug: Data not saving to Firebase (only localStorage)

**Symptoms:**
- Adding players in Admin panel showed "Saved" toast ✅
- Players appeared in dropdown on same device ✅
- Players did NOT appear on other devices/browsers ❌
- Data was only saved locally, not synced to Firebase ❌

**Root Cause:**
The Firebase SDK functions `set` and `get` were imported correctly but never assigned to the `window` object. The `saveData()` function calls `window.__fbSet()` and `loadData()` calls `window.__fbGet()` — both were `undefined`, causing every Firebase write/read to fail silently and fall back to `localStorage`.

**Broken code (before fix):**
```javascript
window.__fbOnValue = onValue;
window.__fbRef = ref;
// ❌ window.__fbSet and window.__fbGet were MISSING
window.__fbReady = false;
window.__db = null;
```

**Fixed code (v1.6.2):**
```javascript
window.__fbOnValue = onValue;
window.__fbRef = ref;
window.__fbSet = set;   // ✅ Added
window.__fbGet = get;   // ✅ Added
window.__fbReady = false;
window.__db = null;
```

**Why it wasn't caught earlier:**
- The app appeared to work normally on a single device
- `showToast('Saved')` fires before the Firebase write attempt
- The `catch(e)` block only logged a `console.warn` — no visible error to the user
- localStorage fallback masked the failure completely

**Impact:**
- All versions from v1.0 to v1.6.1 had this bug
- Any data entered since switching to GitHub Pages was only saved locally
- Data entered on Netlify (before GitHub Pages) was saved correctly because the old Netlify deploy had the correct assignments

**Fix applied in:** v1.6.2
**Files changed:** `index.html` (Firebase init script block)

### Additional debugging steps taken:
1. ✅ Firebase Database Rules confirmed: `.read: true, .write: true`
2. ✅ GitHub Pages domain authorized in Firebase Authentication
3. ✅ Firebase Database URL confirmed correct
4. ✅ Console showed only `favicon.ico 404` (harmless)
5. ✅ Cross-device test confirmed localStorage-only behaviour
6. 🎯 Root cause identified: missing `window.__fbSet` and `window.__fbGet` assignments


---

## 21. v1.7.0 — Photo Gallery Redesign

**Date:** March 2026

### Photo Gallery — Full Redesign

Replaced the old list-only photo view with a modern gallery system:

- **Grid view** (default) — thumbnail tiles with hover overlay showing caption
- **List view** — detailed rows with metadata and action buttons
- **Grid/List toggle** in toolbar
- **Lightbox** — full-screen photo viewer with ← → keyboard navigation, click-outside-to-close
- **Upload modal** — drag & drop multi-file uploader with per-photo captions, progress bar (replaces browser `prompt()`)
- Inline caption editing per photo

**New CSS classes added:** `.photo-grid`, `.lightbox-overlay`, `.upload-modal-overlay`, `.gallery-toolbar`, `.gallery-view-btn`, `.photo-grid-item`, `.photo-grid-overlay`, `.photo-action-btn`, `.lightbox-nav`, `.upload-drop-zone`, `.upload-preview-list`, `.caption-edit-wrap`

---

## 22. v1.8.0–v1.8.2 — Firebase 1MB Node Limit Root Cause Discovery

**Date:** March 2026

### Problem: Players Disappearing After Firebase Write

**Symptoms:**
- Players added in Admin, appeared locally ✅
- After page reload, 1–2 players would vanish ❌
- Firebase Diagnostics showed: writes acknowledged but verify check showed ❌ NOT FOUND
- Isolated `_writetest` path worked fine ✅

**Root Cause:**
Firebase Realtime Database has a **1MB per-node write limit**. The `tsvgd` root node had grown to **~1.8MB** because photo `dataURL` base64 strings were being stored inside Firebase with every `saveData()` call. Firebase silently rejected the oversized writes and reverted to the last valid snapshot — no error thrown to the app.

**Diagnostic output that revealed the issue:**
```
Force save data size: 1848140 chars   ← 1.8MB, over limit
Players write OK ✅                    ← Firebase acknowledges
Verify player in Firebase: ❌ NOT FOUND  ← Firebase reverted it
Isolated write verify: ✅ DATA PERSISTED outside tsvgd
Total tsvgd size: 1804.9 KB           ← Confirmed bloated
```

**Fix (v1.8.1):** Strip `dataURL` and `photoURL` fields from all Firebase writes. Photos remain in `localStorage` only; only metadata (caption, uploader, date, filename) goes to Firebase.

```javascript
delete cleanForFirebase.photos[album][pid].dataURL;
delete cleanForFirebase.profiles[p].photoURL;
```

---

## 23. v1.8.3 — Granular Firebase Writes (Architecture Overhaul)

**Date:** March 2026

### Problem: Monolithic saveData() Writing Everything Every Time

Every action (click Available, save settings, rename player) triggered `saveData()` which serialised and wrote the entire database to Firebase. This caused the 1MB limit to be hit and also created race conditions.

### Solution: `fbWrite(path, value)` — Targeted Path Writes

Introduced a central `fbWrite()` helper that writes **only the specific path that changed**:

```javascript
function fbWrite(path, value){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(removeUndefined(sharedData)));
  var cleanValue = stripDataURLs(value);
  return window.__fbSet(window.__fbRef(window.__db, 'ss2026/'+path), cleanValue);
}
```

**Each action now writes only its own path:**

| Action | Firebase write | Size |
|--------|---------------|------|
| Click Available | `players/Harish Raju/50overs_3` | ~50 bytes |
| Add player | `players/Amit` + `pins/Amit` | ~30 bytes |
| Save settings | `settings` | ~200 bytes |
| Delete player | `players/X` = null | minimal |
| Mark attendance | `practice/id/attendance/name` | ~20 bytes |
| Upload photo | localStorage only | 0 bytes to Firebase |

`saveData()` retained for bulk operations only (backup import/restore) with a 900KB size guard.

---

## 24. v1.8.4–v1.8.9 — Firebase Node Key, Empty Object & PIN Bugs Fixed

**Date:** March 2026

### Issue 1: tsvgd node tombstone (v1.8.6)

After manually deleting the bloated `tsvgd` node in Firebase Console, it was set to `null`. Some Firebase SDK versions treat deleted nodes as tombstones where writes are accepted but immediately reverted.

**Fix:** Changed the root Firebase key from `tsvgd` to `ss2026`. Also updated `localStorage` key from `tsvgd-2026` to `ss2026-data`.

### Issue 2: Firebase deletes empty `{}` objects (v1.8.7)

Firebase Realtime Database **silently prunes empty objects** — writing `{}` to a path causes Firebase to delete that path entirely. When adding a new player, `players/{name}` was written as `{}` and Firebase deleted it immediately, causing `pins` and `tempPins` to appear but `players` to be missing.

**Fix:** Player nodes are now initialised as `{_p: true}` instead of `{}`.

```javascript
// Before (deleted by Firebase)
window.__fbSet(ref('ss2026/players/'+name), {})

// After (preserved by Firebase)  
window.__fbSet(ref('ss2026/players/'+name), {_p: true})
```

### Issue 3: PIN reset sent player straight to "Set PIN" (v1.8.8)

When admin reset a player's PIN, the next login would jump straight to the set-new-PIN screen instead of asking for a temp PIN first. This was because the `tempPins` flag was not being set on reset, and `onConfirmPinDigit()` called `loginSuccess()` before the Firebase write completed — `onValue` would then fire with stale Firebase data (still holding the old hash) and overwrite `sharedData`.

**Fix:** 
1. `resetPlayerPin()` now auto-generates a random 4-digit temp PIN, sets `tempPins[player] = true`, writes both to Firebase, and shows the generated PIN in a modal popup for the admin to share.
2. `onConfirmPinDigit()` now waits for all three Firebase writes (`pins`, `players`, `tempPins`) to complete via `Promise.all().then()` before calling `loginSuccess()`.

### Issue 4: `sharedData.players` undefined crash (v1.8.5)

When Firebase was empty (fresh database), `onValue` fired with `null` and `sharedData` became `{}` (no `players` key). Any function accessing `sharedData.players[name]` crashed with `TypeError: Cannot read properties of undefined`.

**Fix:** Guards added at entry points of `adminAddPlayer()`, `buildCard()`, `renderMatches()`, and the `onValue` handler:
```javascript
if(!sharedData) sharedData = {};
if(!sharedData.players) sharedData.players = {};
if(!sharedData.pins) sharedData.pins = {};
```

---

## 25. v1.8.9 — Mobile Bottom Tab Bar & Auto Temp PIN

**Date:** March 2026

### Mobile Navigation — Bottom Tab Bar

Replaced the mobile dropdown menu with a **fixed bottom tab bar** (Option 1 — native app style):

**Primary tabs (always visible):**
- 🏏 50 Overs
- ⚡ T20  
- 🏋 Practice
- 📊 Overview
- ☰ More

**More drawer** (slides up from bottom bar):
- 🏏 Friendly · 🕐 Activity · 📋 Matches · 👥 Team · 📸 Photos · ⚙️ Admin

**Behaviour:**
- Active tab highlighted in gold
- Pending response badge counts on 50 Overs and T20 tabs
- More button highlighted when a secondary tab is active
- Drawer closes when tapping outside or selecting a tab
- Content area padded to avoid overlap with fixed bar (`padding-bottom: 70px` on mobile)
- Safe area insets handled for iPhone home bar (`env(safe-area-inset-bottom)`)

### PIN Reset — Auto-Generate Temp PIN

`Admin → Reset PIN` now auto-generates a secure random 4-digit temp PIN instead of deleting the PIN entirely:

**Old behaviour:** Reset PIN → delete PIN hash → player goes straight to "Set new PIN" (no temp PIN check)

**New behaviour:** Reset PIN → generate random PIN → write to Firebase as temp PIN → show PIN in modal popup → player enters temp PIN → prompted to set own PIN

**Modal shows:**
```
New temp PIN for [Player Name]
        4 8 2 7
Share this PIN with the player.
They will be prompted to set a new PIN on next login.
[Got it ✓]
```

---

## 26. Current Firebase Data Structure (v1.8.9+)

**Root key changed from `tsvgd` to `ss2026`**  
**localStorage key changed from `tsvgd-2026` to `ss2026-data`**

```
ss2026/
├── players/{playerName}: {_p:true, [league_idx]:{status,reason,notes,lastEdit}}
├── pins/{playerName}: hashString
├── tempPins/{playerName}: true          ← set on add/reset, deleted on PIN change
├── profiles/{playerName}: {jersey, role, batting, bowling, dob, joined, contact, emergency, specialRoles, email}
├── improvements/{playerName}: [improvementId, ...]
├── achievements/{matchKey}/{playerName}/badges: [achievementId, ...]
├── matchSummaries/{matchKey}: {result, ourScore, oppScore, motm, bomm, notes, savedBy, savedAt}
├── scorecards/{matchKey}: {ourInnings:{batting,bowling,extras,total,wkts,overs}, oppInnings:{...}}
├── practice/{sessionId}: {id, date, time, type, location, mapLink, notes, createdBy, responses/{}, attendance/{}}
├── friendly/{matchId}: {id, opponent, date, time, format, homeAway, location, notes, createdBy, responses/{}}
├── photos/{albumKey}/{photoId}: {id, caption, uploadedBy, uploadedAt, fileName, fileSize}
│   NOTE: dataURL stored in localStorage only — never written to Firebase
├── grounds: [{name, address, mapLink}]
├── permissions: {admins:[], canCreateSessions:[], canUploadPhotos:[]}
├── settings: {warnThreshold, defaultGround, defaultGroundIdx, teamName, activeSeason, instagramHandle, theme}
├── schedule/{league_season}/{matchId}: {id, day, date, dateISO, time, league, season, home, away, venue, address, location}
├── seasons: [year, ...]
├── lateCancellations/{playerName}: [{label, time}]
├── adminUnlocks/{league_idx_player}: true
├── lastEdits/{playerName}: ISO timestamp
├── playerRoles/{playerName}: {roleId: 'accepted'|'pending'}
├── roleNotifications/{playerName}: {roleId: 'new'}
├── customRoles: [{id, emoji, label, perms}]
├── customAchievements: [{id, emoji, label}]
├── customImprovements: [{id, emoji, label}]
├── authType/{playerName}: 'pin'|'password'
└── adminUnlocks/{key}: true
```

### Key Architecture Rules (v1.8.9+)
- **Photos:** `dataURL` and `photoURL` (base64) are NEVER written to Firebase — localStorage only
- **Player nodes:** Always initialised as `{_p: true}` — never `{}` (Firebase prunes empty objects)
- **All writes:** Use `fbWrite(path, value)` for targeted single-path writes, not `saveData()`
- **PIN flow:** Admin sets temp PIN → player logs in with temp PIN → forced to set own PIN → `tempPins` flag cleared from Firebase
- **Firebase node size:** Kept well under 1MB limit by excluding base64 data


---

## 27. v1.9.0–v1.9.8 — Mobile UI, Bug Report, Temp PIN Display & CSS Fixes

**Date:** March 2026

### v1.9.0 — Mobile Bottom Tab Bar (live)
- Added `@media(max-width:700px)` query to show `.mobile-tab-bar` and hide `#desktop-tabs`
- Fixed pending reminder chips — date and opponent now separated by ` · `
- Fixed pending reminder banner items — date, separator, team, tag badge all on one clean row

### v1.9.1 — More Drawer Fixed
- `toggleMoreDrawer()` was missing — old `toggleMobileMenu()` still referenced `mobile-tab-menu` (old dropdown)
- `switchTab()` updated to highlight correct bottom bar tab and close drawer

### v1.9.2 — Photos Tab Placeholder
- Photos panel replaced with "Coming Soon" card explaining Firebase Storage is being set up
- Pulsing gold "Work in progress" badge
- Feature preview pills: Match / Practice / Team photos
- Photos tab and More drawer show subtle `soon` label
- All upload/gallery JS preserved — UI is a simple swap back when Firebase Storage is ready

### v1.9.3 — Bug Report System
**Player side (🐛 Bugs tab):**
- Type selector: Bug / UI Issue / Feature Request / Data Issue / Other
- Description textarea + feature/location field
- Submit → saved to Firebase `bugReports/` instantly
- Player sees their own past reports with status and admin notes

**Admin side (Admin panel → 🐛 Bug Reports section):**
- All reports consolidated, newest first, with type emoji and reporter
- Status dropdown per report: open / reviewing / fixed / wontfix (saved to Firebase instantly)
- Admin note per report — visible to the reporter on their Bugs tab
- Delete report, filter by status, refresh button

### v1.9.4 — Temp PIN Visible in Admin Panel
**Problem:** Admin reset PIN showed a splash modal but if dismissed, the PIN was lost.

**Fix:** Temp PIN now stored in `tempPinDisplay/{playerName}: {pin, setAt}` in Firebase.

- Player row in admin shows: `🔑 Temp PIN · 4827 (2 mins ago)`
- PIN number visible until player sets their own — then automatically cleared
- Works for both Add Player and Reset PIN flows
- Cleared in Firebase when: `onConfirmPinDigit()`, `saveNewAuth()`, `handleSetPassword()`

### v1.9.5 — Matches Tab & Bug Form Fixes
- Bug report textarea made full width with `min-height:120px`
- `initScheduleFromHardcoded()` now copies `address` and `mapLink` fields from hardcoded MATCHES
- Migration flag bumped to `scheduleInitialisedV2` to force re-copy for existing users
- Upcoming match banner redesigned: `⏳ Upcoming  ✓ N  ? N  ✗ N  responded`

### v1.9.6–v1.9.8 — Missing CSS Discovery
**Root cause:** CSS class definitions were being lost across rebuilds. Over 80 classes were missing.

**Added in v1.9.6:** `.match-result-card`, `.match-result-header`, `.match-result-body`, `.match-upcoming-banner`, `.result-badge` variants

**Added in v1.9.7:** All Team tab CSS — `.player-card`, `.player-card-header`, `.player-avatar`, `.player-avatar-placeholder`, `.player-card-name/role/body`, `.special-badges`, `.achievements-section`, `.achievement-badge`, `.improvement-badge`, `.player-expanded`, `.expand-btn`, `.career-stats-grid`

**Added in v1.9.8:** Practice cards, friendly cards, scorecard tables, standings table, attendance grid, role badges, pending chips, section dividers, match card extras, overview grid, activity feed rows, export buttons, photo gallery, upload modal, league toggle

---

## 28. v2.0.0 — Major CSS Restoration & Bug Fixes

**Date:** March 2026

### Root Cause of All Visual Issues

The most fundamental CSS classes had been lost — `.form-input`, `.form-label`, `.form-field`, `.form-grid` — causing every form, input, label and modal across the entire app to render incorrectly (white backgrounds, inline labels, broken layouts).

### All Fixes Applied

| # | Bug | Fix |
|---|-----|-----|
| 1 | All forms: white input backgrounds | `.form-input` — dark theme `var(--deep)` background, focus ring |
| 2 | All forms: labels inline with inputs | `.form-label` + `.form-field` — flex column, label above input |
| 3 | All forms: no grid layout | `.form-grid` — `repeat(auto-fit, minmax(220px, 1fr))` |
| 4 | Tab badges merging with label text `50 OVERS9` | Tab button text wrapped in `<span>`, badge appended separately |
| 5 | Overall Response progress bar invisible | `.progress-bar`, `.progress-fill`, `.progress-pct` restored |
| 6 | Pending reminder banner: no background | `.login-reminder` — `var(--mid)` background, border, rounded corners |
| 7 | Activity feed: no card styling | `.act-feed-row` — background, border, hover state |
| 8 | Away icon was 🔵, Home was 🟢 | Away changed to 🔴 across all tabs |
| 9 | Home team name missing in Matches tab | Fixed `buildMatchResultCard` — shows `TSV Großdeinbach vs Opponent` |
| 10 | Edit Profile visible to all players | Guard: only shown when `player === currentPlayer \|\| adminUnlocked` |
| 11 | Edit Match modal: labels inline | Fixed to use `.form-grid` with labels above inputs; fixed `id="am-address"` |
| 12 | Add Ground form: labels inline | Confirmed using `.form-grid` — now renders correctly with CSS restored |

### Also restored in v2.0.0
`.btn-save`, `.btn-cancel`, `.session-form`, `.form-actions`, `.attendance-*`, `.league-toggle`, `.ltoggle`, `.admin-btn` variants, `.admin-action-btn` variants, `.role-pending`, `.export-btn`, `.add-session-btn`, `.practice-toolbar`, `.month-nav-btn`, `.edit-match-btn`, `.offline-bar`, `.saving-badge`

---

## 29. Current Version Summary (v2.0.0)

**App version:** v2.0.0  
**Firebase root key:** `ss2026`  
**localStorage key:** `ss2026-data`  
**GitHub Pages:** schwabensentinelstsv1897-alt.github.io/Schwaben-sentinels/

### Tabs
| Tab | Status | Notes |
|-----|--------|-------|
| 50 Overs | ✅ Live | Availability, lock rule, team overview |
| T20 | ✅ Live | Same as 50 Overs |
| Friendly | ✅ Live | Admin/permitted players create matches |
| Practice | ✅ Live | Sessions + Attendance grid |
| Overview | ✅ Live | Full grid, Excel/PDF export |
| Activity | ✅ Live | Compact feed, click to expand |
| Matches | ✅ Live | Results, scorecards, AI import, standings |
| Team | ✅ Live | Player cards, roles, achievements, career stats |
| Photos | 🔄 Coming soon | Firebase Storage integration pending |
| 🐛 Bugs | ✅ Live | Player reports → Admin panel consolidated view |
| Admin | ✅ Live | Players, roles, settings, bug reports, backup |

### Mobile Navigation (≤700px)
Bottom tab bar with 5 primary tabs + More drawer:
- Primary: 🏏 50 Overs · ⚡ T20 · 🏋 Practice · 📊 Overview · ☰ More
- More drawer: Friendly · Activity · Matches · Team · Photos · Bugs · Admin

### Known Pending Items
- 📸 Firebase Storage for Photos tab
- 📊 Player Stats tab (career batting/bowling breakdown)
- 🏆 Season Leaderboard

---

## 30. v2.0.1–v2.0.5 — Bug Fixes, Help Tab & Rollout

**Date:** March 2026

### v2.0.1 — Bug Report Functions Restored
Bug report JS functions (`submitBugReport`, `renderMyBugReports`, `renderAdminBugReports`, `updateBugStatus`, `saveAdminNote`, `deleteBugReport`) were dropped during the v2.0.0 CSS rebuild and had to be restored.

**Data flow:**
- Player submits → Firebase `ss2026/bugReports/{id}` → visible in player's past reports list
- Admin opens Admin panel → Bug Reports section → sees all reports with status dropdown and note field
- Status (`open / reviewing / fixed / wontfix`) and admin notes write to Firebase instantly
- Players see admin notes on their own Bugs tab

### v2.0.2 — Settings Form Not Loading Saved Values
`loadSettingsIntoForm()` was only populating 2 of 6 fields (`warnThreshold`, `defaultGround`). All other fields were blank on every open.

**Fixed to load all fields:**
- Team Name → `setting-team-name`
- Active Season → `setting-season`
- Instagram Handle → `setting-instagram`
- Colour Theme → `setting-theme`
- Default Practice Ground → matches by `defaultGroundIdx` first, falls back to name match

### v2.0.3 — Help Tab Added
New **❓ Help** tab in desktop bar and More drawer on mobile. Contains the full player guide styled in the app theme:
- Numbered first-login steps with green step circles
- Colour-coded availability cards (green/amber/red)
- Tab reference list
- Profile setup instructions
- Tips section

Also generated `schwaben-sentinels-guide.pdf` — printable A4 guide with:
- Dark green header on every page with club logo, SCHWABEN SENTINELS title, and website URL
- Colour-coded tables for availability, tabs, steps
- Page numbers in footer

### v2.0.4 — UI Cleanup
- **Share bar removed** — "Share this page with your players" green banner removed from main panel
- **Default PIN removed** — "Default admin PIN: 1897" note removed from admin lock screen

### v2.0.5 — Mobile UX & Notification Fixes

**Fix 1 — More drawer ghost on mobile:**
Replaced `toggleMoreDrawer()` in all drawer buttons with `closeMoreDrawer()` — a dedicated close-only function. Added to `switchTab()` as well. The drawer now closes instantly on tap without lingering.

**Fix 2 — Help tab logo:**
Replaced the 🏏 bat emoji in the Help tab header card with the actual club logo image — copied from the header `<img>` when the Help tab is opened.

**Fix 3 — Friendly tab badge:**
`updateTabBadges()` previously only counted pending responses for 50 Overs and T20. Added Friendly pending count — checks upcoming friendly matches where the current player has not responded. Badge shows on both desktop tab and mobile More drawer item.

**Fix 4 — Grouped pending reminder:**
The login reminder banner was listing every single pending match as a separate row — if a player had 20 pending matches the list was enormous, pushing other content off screen.

New behaviour: pending items are **grouped by league**:
- If only 1 pending in a league → shows as before (date · opponent · tag)
- If 2+ pending in a league → collapsed to `50 Overs · 5 pending ▼`
- Click the row to expand and see all individual matches
- Arrow rotates on expand/collapse
- `toggleReminderGroup(id)` function handles expand/collapse

---

## 31. First Team Rollout (v2.0.5)

**Date:** March 2026

### Rollout Checklist (Admin)
1. Upload `schwaben-sentinels-v2.0.5.html` to GitHub as `index.html`
2. Add all players via Admin → Add Player with temp PINs
3. Share the live URL with each player: `schwabensentinelstsv1897-alt.github.io/Schwaben-sentinels/`
4. Share their individual temp PIN (visible in Admin panel next to their name)
5. Send the rollout message (see `schwaben-sentinels-rollout.md`)

### Players Added at Rollout
- Harish Raju
- Narendra Reddy Bakki
- Niranjan Gorla
- Pradeep Prabhakara (admin)
- Surendra Malapati
- Avin Mathew
- Amit Tumdi

### Known Pending Items at Rollout
- 📸 Firebase Storage for Photos tab (currently shows "coming soon")
- 📊 Player Stats tab (career batting/bowling breakdown)
- 🏆 Season Leaderboard

