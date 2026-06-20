# ChatCircle - Private Group Chat App

## Product Requirements Document (PRD)

---

## Document Information

| Property | Value |
|----------|-------|
| **Product Name** | ChatCircle |
| **Document Version** | 1.0 |
| **Date** | June 2026 |
| **Platform** | Android (Flutter) |
| **Backend** | Appwrite Cloud |
| **Notifications** | FCM + OneSignal |

---

## 1. Executive Summary

### 1.1 Product Vision

ChatCircle is a private, invite-only group chat application designed for small, close-knit circles of 6-7 friends. The app prioritizes privacy, controlled access, and a premium user experience with cutting-edge UI design. Unlike traditional messaging apps, ChatCircle eliminates direct messaging and focuses entirely on group-based communication under admin governance.

### 1.2 Problem Statement

Existing messaging apps (WhatsApp, Telegram, Signal) are designed for mass communication with open group creation, DMs, and complex permission systems. Small friend groups need a dedicated space where:
- Communication stays within the circle
- An admin has full control over membership
- Content is ephemeral and manageable
- The experience feels premium and intentional

### 1.3 Target Audience

- **Primary:** Groups of 6-7 close friends (age 18-35)
- **Secondary:** Small teams, family groups, study circles
- **Tertiary:** Communities requiring strict admin governance

---

## 2. Core Concept & Philosophy

### 2.1 Fundamental Principles

| Principle | Description |
|-----------|-------------|
| **Privacy First** | No DMs, no data mining, no unnecessary permissions |
| **Admin Governance** | One admin controls all aspects of the app |
| **Intentional Design** | Every feature serves a purpose; no bloat |
| **Ephemeral Content** | Messages and stories auto-delete |
| **Premium Experience** | Glassmorphism + fluid animations |

### 2.2 Key Differentiators

| Feature | ChatCircle | WhatsApp | Telegram | Signal |
|---------|------------|----------|----------|--------|
| No DMs | ✅ | ❌ | ❌ | ❌ |
| Admin-Only Groups | ✅ | ❌ | ❌ | ❌ |
| Auto-Delete Messages | ✅ | ⚠️ | ✅ | ✅ |
| Stories (Admin Controlled) | ✅ | ✅ | ❌ | ❌ |
| Biometric App Lock | ✅ | ❌ | ❌ | ❌ |
| Mandatory Splash Alerts | ✅ | ❌ | ❌ | ❌ |
| Force Update Lock | ✅ | ❌ | ❌ | ❌ |
| Glassmorphism UI | ✅ | ❌ | ❌ | ❌ |

---

## 3. User Roles & Permissions

### 3.1 Role Definitions

| Role | Count | Permissions |
|------|-------|-------------|
| **Admin** | 1 (max) | Full system control, user approval, group management, message deletion |
| **Approved User** | 5-6 | Chat in groups they're added to, view stories, post stories (if permitted) |
| **Pending User** | Variable | Account created, awaiting admin approval - cannot login |
| **Blocked User** | Variable | Cannot login; effectively banned |

### 3.2 Permission Matrix

| Action | Admin | Approved User | Pending | Blocked |
|--------|-------|---------------|---------|---------|
| Login | ✅ | ✅ | ❌ | ❌ |
| View Default Group | ✅ | ✅ | ❌ | ❌ |
| View Added Groups | ✅ | ✅ (if added) | ❌ | ❌ |
| Send Messages | ✅ | ✅ (in added groups) | ❌ | ❌ |
| Post Stories | ✅ | ✅ (if permitted) | ❌ | ❌ |
| View Stories | ✅ | ✅ | ❌ | ❌ |
| Create Groups | ✅ | ❌ | ❌ | ❌ |
| Approve Users | ✅ | ❌ | ❌ | ❌ |
| Delete Messages | ✅ | ❌ | ❌ | ❌ |
| Ban Users | ✅ | ❌ | ❌ | ❌ |

---

## 4. User Onboarding & Authentication

### 4.1 Sign-Up Flow

```
┌─────────────────────────────────────────────────────────┐
│                    SIGN-UP SCREEN                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │  ✉️ Email Address                              │   │
│  │  🔒 Password (min 8 chars)                    │   │
│  │  👤 Full Name                                 │   │
│  │  [📸 Upload Avatar] (optional)                │   │
│  │                                               │   │
│  │         [ CREATE ACCOUNT ]                     │   │
│  │                                               │   │
│  │  "Already have an account? Sign In"           │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Glassmorphic Card | Gradient Background              │
└─────────────────────────────────────────────────────────┘
```

**Flow:**
1. User enters email, password, name
2. Avatar upload (optional) - stored in Appwrite Storage
3. Account created in Appwrite Auth with `status = 'pending'`
4. Document created in `users` collection
5. Admin receives real-time notification in dashboard
6. Toast: *"Account created – waiting for admin approval"* (glassmorphic)

### 4.2 Login Flow

```
┌─────────────────────────────────────────────────────────┐
│                     LOGIN SCREEN                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │  ✉️ Email Address                              │   │
│  │  🔒 Password                                   │   │
│  │  [👆 Use Biometric] (if enabled)               │   │
│  │                                               │   │
│  │         [ SIGN IN ]                            │   │
│  │                                               │   │
│  │  "Don't have an account? Sign Up"             │   │
│  │  "Forgot Password?"                           │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Glassmorphic Card | Gradient Background              │
└─────────────────────────────────────────────────────────┘
```

**Status Checks:**
| Status | Login Result |
|--------|--------------|
| `approved` | ✅ Login successful → Home Screen |
| `pending` | ❌ Toast: *"Admin approval pending"* |
| `blocked` | ❌ Toast: *"Account suspended"* |

### 4.3 App Lock / Biometric Authentication

After successful login, users are prompted to enable App Lock.

**Features:**
- Fingerprint/Face unlock support (`local_auth`)
- 4-6 digit PIN fallback
- Auto-lock after 5 minutes inactivity (configurable)
- Settings page to enable/disable
- Admin can force-enable via remote config

**Lock Screen UI:**
- Minimal glassmorphic design
- App logo centered
- Biometric icon + PIN entry
- "Forgot PIN?" recovery option (email verification)

---

## 5. Chat & Messaging

### 5.1 Group Structure

```
┌─────────────────────────────────────────────────────────┐
│                    CHAT SCREEN                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │ 🔙 [Group Name]         👥 6   ⚙️             │   │
│  ├─────────────────────────────────────────────────┤   │
│  │  📌 [Pinned Broadcast Message] (Admin only)    │   │
│  ├─────────────────────────────────────────────────┤   │
│  │                                                 │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │  User A  12:30                          │  │   │
│  │  │  Hey everyone!                         │  │   │
│  │  │  ❤️ 2  👍 1                           │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  │                                                 │   │
│  │       ┌────────────────────────────────────────┐ │   │
│  │       │  User B  12:32                       │ │   │
│  │       │  I'm in! 🔥                        │ │   │
│  │       │  ❤️ 1  😂 3                       │ │   │
│  │       └────────────────────────────────────────┘ │   │
│  │                                                 │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │  User A is typing...                    │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  ├─────────────────────────────────────────────────┤   │
│  │  [📎]  [Type a message...]  [📤]  [😊]       │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Glassmorphic Chat Bubbles | Adaptive Transparency    │
└─────────────────────────────────────────────────────────┘
```

### 5.2 Messaging Rules

| Rule | Description |
|------|-------------|
| **No DMs** | Users cannot send private messages |
| **Group Only** | All messages go to a group |
| **Default Group** | All approved users are in the default group |
| **Group Visibility** | Users only see groups where `allowed_users` contains their UID |
| **Real-Time** | Appwrite Realtime subscriptions for instant updates |

### 5.3 Message Features (Long-Press)

Long-press on any message opens a glassmorphic pop-up with:

| Action | Description | Visibility |
|--------|-------------|------------|
| **❤️ 👍 🔥 😂 😮 😢** | iOS-style emoji reactions | All users |
| **Reply** | Quoted reply to specific message | All users |
| **Forward** | Forward to another group | Admin only |
| **Copy** | Copy text to clipboard | All users |
| **Delete** | Delete message | Admin only |

### 5.4 Typing Indicator

- Real-time display of who is typing
- Uses Appwrite Realtime with `typingUsers` map
- Auto-clears after 3 seconds of inactivity
- Shows as "User A is typing..." below messages

### 5.5 Read Receipts (Double Tick)

| Status | Icon | Description |
|--------|------|-------------|
| **Sent** | ✓ | Message sent to server |
| **Delivered** | ✓✓ | Delivered to all group members |
| **Read** | ✓✓ (blue) | Read by all group members |

---

## 6. Stories (Status) Section

### 6.1 Layout

```
┌─────────────────────────────────────────────────────────┐
│  📸 Your Story   👤 Friend 1   👤 Friend 2   👤 +3   │
│  ───────────────────────────────────────────────────── │
│  [Horizontal Scroll - Like WhatsApp/Telegram]          │
└─────────────────────────────────────────────────────────┘
```

### 6.2 Story Features

| Feature | Description |
|---------|-------------|
| **Content** | Text-only initially (images later) |
| **Duration** | Auto-deletes after 24 hours |
| **Posting Permission** | Admin-controlled whitelist |
| **Viewing Permission** | All approved users |
| **Master Toggle** | Admin can disable stories entirely |
| **Expiry** | Appwrite Scheduled Function deletes expired stories |

### 6.3 Story Posting UI

```
┌─────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────────────────────────┐   │
│  │  📝 Type your story...                         │   │
│  │                                                 │   │
│  │  [📸 Add Image] (future)                      │   │
│  │                                                 │   │
│  │         [ POST STORY ]                          │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Appears at bottom when "Your Story" is tapped        │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Admin Panel

### 7.1 Access & Security

- Separate admin route: `/admin` in app navigation
- Only users with `role == 'admin'` can access
- Optional 2FA (authenticator app) for admin
- Session timeout: 30 minutes inactivity

### 7.2 Dashboard Overview

```
┌─────────────────────────────────────────────────────────┐
│  ⚙️ ADMIN DASHBOARD                                    │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐            │
│  │ 12   │  │ 3    │  │ 2    │  │ 5    │            │
│  │Users │  │Pending│  │Groups│  │Stories│           │
│  └──────┘  └──────┘  └──────┘  └──────┘            │
│                                                         │
│  [Pending Approvals] [Group Management]               │
│  [User Control] [Story Settings]                      │
│  [Auto-Clean] [Broadcast Pin]                        │
│  [System Alerts] [Force Update]                      │
└─────────────────────────────────────────────────────────┘
```

### 7.3 Pending Approvals

| Column | Action |
|--------|--------|
| User Name | Display |
| Email | Display |
| Signup Date | Display |
| **Approve** | ✅ Status → `approved`, FCM notification sent |
| **Reject** | ❌ Status → `blocked` |

### 7.4 Group Management

**Create Group:**
- Name (required)
- Avatar (optional)
- Auto-delete timer (24h / 7d / Off)

**Group Details Screen:**
- List all users with toggle switch
- ON → User UID added to `allowed_users`
- OFF → User removed from `allowed_users`
- Group instantly appears/disappears on user's app

### 7.5 User Audit & Control

| Feature | Description |
|---------|-------------|
| **Full User List** | All registered users with details |
| **Change Role** | `user` ↔ `admin` |
| **Ban/Unban** | Set `status = 'blocked'` |
| **Delete Account** | Remove from Auth + Firestore |

### 7.6 Story Settings

| Setting | Description |
|---------|-------------|
| **Master Toggle** | Enable/Disable Stories globally |
| **Poster Whitelist** | Add/remove users who can post stories |
| **View All** | All approved users can view (non-configurable) |

### 7.7 Auto-Clean / Self-Destruct

| Setting | Options |
|---------|---------|
| **Per Group** | 24 hours / 7 days / Off |
| **Admin Toggle** | Turn ON/OFF per group |
| **Implementation** | Appwrite Scheduled Function checks `timestamp` |

### 7.8 Admin Broadcast Pin

- Pin a message to the top of chat header
- Glassmorphic card with admin name + timestamp
- Remains until admin removes it
- Supports formatted text (bold, italic, links)

### 7.9 System Alert Manager (Entry Gate)

**Purpose:** Show a mandatory popup every time any user opens the app.

| Field | Description |
|-------|-------------|
| **Title** | Alert title (e.g., "Important Notice") |
| **Message** | Multi-line text content |
| **Buttons** | "Exit" (closes app) / "Acknowledge" (dismisses) |
| **Deactivate** | Removes alert completely |

**Technical:**
- Stored in `app_settings` collection
- `updatedAt` timestamp compared locally
- Full-screen glassmorphic overlay

### 7.10 Force Update Manager

**Purpose:** Enforce minimum version code for all users.

| Field | Description |
|-------|-------------|
| **Minimum Version Code** | Integer (e.g., 12) |
| **Update Message** | Optional custom message |
| **Save** | Updates `app_settings` collection |

**Blocking Screen:**
- Full-screen, no bypass
- "Update Now" → Play Store link
- No "Exit" or "Skip"

---

## 8. Notification System

### 8.1 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  NOTIFICATION ARCHITECTURE              │
│                                                         │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐  │
│  │  Appwrite  │───▶│  OneSignal │───▶│    FCM     │  │
│  │  Triggers  │    │  (Gateway) │    │  (Google)  │  │
│  └────────────┘    └────────────┘    └────────────┘  │
│                                                         │
│  Events:                                               │
│  • User approved/rejected                              │
│  • Added/removed from group                           │
│  • New message (background)                           │
│  • New story posted                                   │
└─────────────────────────────────────────────────────────┘
```

### 8.2 Notification Events

| Event | Trigger | Recipient |
|-------|---------|-----------|
| **Approval** | Admin approves user | Approved user |
| **Rejection** | Admin rejects user | Rejected user |
| **Group Added** | User added to group | Added user |
| **Group Removed** | User removed from group | Removed user |
| **New Message** | Message sent in group | All group members (background) |
| **New Story** | Story posted | All approved users |
| **System Alert** | Admin publishes alert | All users |

### 8.3 In-App Foreground Notifications

**Behavior when app is open:**
- ❌ Do NOT show system notification
- ✅ Show glassmorphic toast overlay
- Sliding animation from top/bottom
- Displays sender + message preview + timestamp
- Tap → navigate to group
- Auto-dismisses after 4-5 seconds

**Implementation:**
- OneSignal `notificationWillShowInForeground` handler
- Custom Flutter UI overlay

### 8.4 Notification Center

- Bell icon in app bar
- List of last 50 notifications
- Read/unread status
- Tap to navigate to relevant screen

---

## 9. UI/UX Design System

### 9.1 Visual Style

| Style | Application |
|-------|-------------|
| **Glassmorphism** | Primary UI containers, cards, toasts |
| **Neumorphism** | Subtle soft shadows on buttons (sparingly) |
| **Liquid Glass** | Dynamic blur intensity, smooth transitions |
| **Adaptive Transparency** | Context-aware transparency |
| **Generative Ambient UI** | Subtle gradient shifts, particle effects |

### 9.2 Color Palette

```
Primary Background:  #0A0A0F (Deep Dark)
Secondary Background: #1A1A2E (Dark Purple)
Glass Base:          rgba(255,255,255,0.05)
Glass Border:        rgba(255,255,255,0.08)
Accent Primary:      #6C63FF (Violet)
Accent Secondary:    #FF6B9D (Pink)
Accent Tertiary:     #00D4FF (Cyan)
Text Primary:        #FFFFFF
Text Secondary:      rgba(255,255,255,0.7)
Text Muted:          rgba(255,255,255,0.4)
```

### 9.3 Typography

**AI-Selected Font Pairing:**

| Role | Font | Style |
|------|------|-------|
| **Heading/Display** | `Poppins` | SemiBold, 18-24sp |
| **Body/Chat** | `Inter` | Regular, 14-16sp |
| **Caption** | `Inter` | Light, 11-12sp |
| **App Bar** | `Poppins` | Medium, 20sp |

**Justification:**
- **Poppins:** Geometric, modern, excellent for titles and headings. Complements glassmorphism with clean lines.
- **Inter:** Highly legible on mobile screens, optimized for reading long messages. Pairs well with Poppins for visual hierarchy.
- Both are open-source, available via `google_fonts` package, and bundled offline.

### 9.4 Key UI Components

| Component | Style |
|-----------|-------|
| **Login/Signup Card** | Glassmorphic, centered, gradient background |
| **Chat Bubbles** | Glassmorphic, sent/received color distinction |
| **Toast Alerts** | Glassmorphic, slide in/out, auto-dismiss |
| **Story Bar** | Horizontal scroll, glassmorphic avatar rings |
| **Admin Toggles** | Neumorphic-inspired switches |
| **Reaction Popup** | Glassmorphic, blurred background |
| **Lock Screen** | Minimal glassmorphic, centered |

### 9.5 Animation Framework

**Flutter Animation Stack:**

| Type | Package/Approach | Use Case |
|------|------------------|----------|
| **Implicit** | `AnimatedContainer`, `AnimatedOpacity` | Toast alerts, story bar, loading states |
| **Explicit** | `AnimationController` + `Tween` | Liquid glass border shifts, story progress rings |
| **Spring/Physics** | `SpringSimulation` | Reaction popups, long-press menu, story dismiss |
| **Hero** | `Hero` widget | Story avatar → detail, group profile picture |
| **Staggered** | `StaggeredAnimation` | Chat messages appearing, story bar cascade |
| **Custom Shader** | `ShaderMask` + `AnimationController` | Generative ambient background, liquid glass ripple |
| **Micro-Interactions** | `flutter_animate` | Every toast, button click, reaction, message send |

**Animation Duration Standards:**
- Toast: 300ms entry, 100ms exit
- Page Transitions: 400ms (fade + scale)
- Button Press: 150ms
- Message Send: 200ms
- Story Bar Item: 150ms delay between items

---

## 10. Appwrite Database Schema

### 10.1 Collections

#### `users`

| Field | Type | Description |
|-------|------|-------------|
| `uid` | String | Appwrite Auth UID (primary) |
| `email` | String | User email |
| `name` | String | Display name |
| `avatar` | String | Avatar URL (Appwrite Storage) |
| `status` | String | `pending` / `approved` / `blocked` |
| `role` | String | `user` / `admin` |
| `isOnline` | Boolean | Online status |
| `lastSeen` | DateTime | Last activity timestamp |
| `fcmTokens` | Array<String> | Multiple device tokens |
| `createdAt` | DateTime | Account creation date |
| `appLockEnabled` | Boolean | Biometric/PIN lock enabled |
| `appLockPIN` | String | Encrypted PIN (flutter_secure_storage) |

#### `groups`

| Field | Type | Description |
|-------|------|-------------|
| `groupId` | String | Auto-generated ID |
| `name` | String | Group name |
| `avatar` | String | Group avatar URL |
| `createdBy` | String | Admin U
