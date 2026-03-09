---
author: muju
title: "Nextcloud Sync"
description: "Integrate Google Services with Nextcloud"
categories: ["Linux"]
tags: ["Nextcloud", "Google"]
image:
  path: /assets/dev/ansible/ansible.webp
  lqip: data:image/webp;base64,UklGRqQAAABXRUJQVlA4IJgAAACwBACdASoUABQAPpFCnEolo6KhqAgAsBIJaQAAMX/3mfYxnydDEyXPI0tcDoS4AP78AqN1nJf6CirU/UZ/cwnvKUYrMaCy5tA2eu7urPBvz/7rGvAoLfqvmhItXbijWuwPZK5sPyrHFXz9qIZTO9qNBuKK+Lo51W0Sl3aJb7xKKzhwEF1C6ZSr+UUtkYmCtzIIm6kwd8AAAA==
published: true
mermaid: false
hidden: false
toc: true
---

# How to Integrate Google Services with Nextcloud: Sync Calendar, Contacts & More (2026 Guide)

![Google to Nextcloud migration header - conceptual image of data flowing from Google icons to Nextcloud logo](https://images.unsplash.com/photo-1516321310764-8a2380f89148?auto=format&fit=crop&q=80&w=1200)  
*Moving away from Google silos or just want a unified view in your self-hosted Nextcloud? Let's set it up!*

Nextcloud is a fantastic open-source alternative for files, calendars, contacts, and more. While there is **no perfect real-time bidirectional sync** with Google services (due to API limitations and privacy choices), the official **Google Integration** app (and its improved fork **Google Synchronization**) allows you to:

- Import/sync **Google Calendar** events (one-way → Nextcloud, with periodic background sync in the fork)
- Import **Google Contacts**
- Import **Google Photos**
- (Partially) access/migrate **Google Drive** files

In this guide we'll focus mostly on **Calendar** and **Contacts** sync — the most requested features.

## Prerequisites

- A running **Nextcloud** instance (version 28 or newer recommended — works great on 29/30 in 2026)
- Admin access to install apps
- A **Google account**
- Access to the [Google Cloud Console](https://console.cloud.google.com/) (you'll need to create OAuth credentials)

## Option 1: Recommended → Use the "Google Synchronization" App (Best for Calendar Background Sync)

The original **integration_google** app is still available, but many users in 2025–2026 prefer the maintained fork:

**nextcloud_google_synchronization** by MarcelRobitaille  
→ GitHub: https://github.com/MarcelRobitaille/nextcloud_google_synchronization

### Step 1: Install the App

1. Go to **Nextcloud → Apps** (top-right menu)
2. Switch to **Tools** category (or search for "Google")
3. If not visible → add custom app repo (recommended):
   - Custom apps → Add app bundle
   - Repository: `https://github.com/MarcelRobitaille/nextcloud_google_synchronization/releases/latest/download/integration_google.tar.gz`
4. Enable **Google Synchronization**

### Step 2: Create OAuth Credentials in Google Cloud Console

1. Go to https://console.cloud.google.com/apis/credentials
2. Create new project (or use existing)
3. Enable APIs:
   - Google Calendar API
   - Google People API (for contacts)
   - Google Photos Library API (optional)
   - Google Drive API (optional)
4. Create **OAuth 2.0 Client IDs** → Web application
5. Authorized redirect URIs (very important!): https://your-nextcloud-domain.com/index.php/apps/integration_google/oauth-redirect
6. Download/save **Client ID** and **Client Secret**

### Step 3: Connect in Nextcloud

1. Go to **Settings → Google Synchronization** (under Administration or Personal — depending on version)
2. Enter your **Client ID** and **Client Secret**
3. Click **Connect to Google** → authorize scopes
4. After successful connection → you'll see your Google calendars, contacts, etc.

### Step 4: Sync Calendar (the Killer Feature)

- Unlike the original app, this fork runs **periodic background jobs**
- Select calendars you want → click **Sync calendar** next to each
- Events from Google → imported into Nextcloud (one-way!)
- New/changed Google events appear after the cron interval (usually 15–60 min)

> **Pro tip**: If you only need read-only calendar view → simpler alternative is subscribing via private ICS link from Google Calendar → add as subscription calendar in Nextcloud (no app needed!).

### Step 5: Import Contacts & Photos

- In the same settings page → buttons for **Import contacts** and **Import photos**
- Contacts → imported into Nextcloud Contacts app
- Photos → usually into a dedicated album/folder

## Option 2: Classic Method – Manual Export / Import (No API Keys Needed)

Good for one-time migration:

1. **Google Takeout** → https://takeout.google.com
- Select Calendar (.ics) + Contacts (.vcf)
2. In **Nextcloud Calendar** → Settings & import → **Import calendar** → upload .ics
3. In **Nextcloud Contacts** → Settings → **Import contacts** → upload .vcf

## Comparison Table: Sync Methods

| Method                          | Calendar Sync      | Background/Auto | Two-way | Complexity | Recommended 2026? |
|---------------------------------|--------------------|------------------|---------|------------|-------------------|
| Google Synchronization fork     | Yes (Google→NC)   | Yes             | No      | Medium     | ★★★★★            |
| Original Google Integration     | Manual import      | No              | No      | Medium     | ★★★              |
| ICS subscription (read-only)    | Yes (Google→NC)   | Almost real-time| No      | Very easy  | ★★★★             |
| Full two-way (custom scripts)   | Possible           | Possible        | Yes     | Very hard  | Only experts     |

## Important Notes & Limitations (2026)

- Most solutions are **one-way** (Google → Nextcloud)
- Google frequently changes API rules — check GitHub issues if connection breaks
- For **Google Drive** true sync → consider **rclone** (server-side) or external storage mount (limited)
- Want full de-Googling? → Migrate completely to Nextcloud + DAVx⁵ (Android) or built-in clients

Have you tried this setup? Did it work smoothly on Nextcloud 30+? Drop your experience in the comments!

Stay sovereign! ☁️✊

*Last updated: January 2026*