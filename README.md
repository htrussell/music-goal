# SiriusXM Pandora Goal Mode — AI Audio Prototype

A browser-based prototype demonstrating a next-generation **Goal-Based Audio Listening System** for SiriusXM Pandora. 

Goal Mode allows listeners to define natural-language listening intentions with hard duration constraints (1–120 minutes) and strict topic exclusions. The system immediately overrides habitual collaborative filtering without retraining foundational recommender models.

---

## Table of Contents
1. [Overview & Core Capabilities](#overview--core-capabilities)
2. [Prerequisites & Obtaining API Keys](#prerequisites--obtaining-api-keys)
   - [1. Google Gemini API Key](#1-google-gemini-api-key)
   - [2. YouTube Data API Key or Bearer Token](#2-youtube-data-api-key-or-bearer-token)
   - [3. Last.fm API Key](#3-lastfm-api-key)
   - [4. Finding a YouTube Playlist ID or URL](#4-finding-a-youtube-playlist-id-or-url)
3. [How to Run the Prototype](#how-to-run-the-prototype)
4. [Step-by-Step Usage Guide](#step-by-step-usage-guide)
5. [Key Architectural Mechanisms](#key-architectural-mechanisms)
6. [Troubleshooting & FAQs](#troubleshooting--faqs)

---

## Overview & Core Capabilities

Traditional audio recommendation algorithms rely heavily on past behavior (collaborative filtering). When a listener wants to switch gears—such as replacing serialized fiction podcasts with sports and economics, or exploring a specific new style or soundtrack canon—standard algorithms keep serving familiar habits.

### What This Prototype Demonstrates:
* **Natural-Language Goal Parsing**: Extracts normalized goals, preferred topics, and hard negative exclusions from freeform user prompts (up to 500 characters).
* **Exact Duration Budgeting**: Enforces strict listening durations (1–120 minutes) using multi-stage candidate packing so audio queues never exceed the listener's available time.
* **Deterministic Exclusions**: Code-level guards guarantee that excluded topics, genres, or artists never enter the generated queue.
* **Ground-Truth Preference Ingestion**: Ingests actual YouTube playlists as a seed for user taste.
* **Prompt-Driven Canon Expansion**: When a listener requests a specific franchise, soundtrack, or artist, Gemini identifies canonical tracks and injects them directly into the candidate recommendations pool.
* **Automated Budget Filling**: If candidate pool items leave an unmet duration deficit, Gemini curates supplemental matching tracks to fill out the requested time.
* **Multi-Tier Album Artwork Resolver**: Pulls high-resolution cover art from YouTube and Last.fm.
* **Automated Gemini Model Failover**: Gracefully recovers from temporary capacity constraints (`503` / `429` / high demand) by cycling through a reliable fallback chain (`gemini-2.5-flash`, `gemini-2.0-flash`, `gemini-1.5-flash`, `gemini-2.0-flash-lite`, `gemini-1.5-flash-8b`).

---

## Prerequisites & Obtaining API Keys

The prototype runs 100% client-side in your browser. All API keys remain strictly in browser session memory and are never transmitted to any third-party backend.

```
┌─────────────────────────────────────────────────────────────────┐
│                     REQUIRED CREDENTIALS                        │
├─────────────────────────┬───────────────────────────────────────┤
│ Google Gemini API Key   │ Structured extraction, budget filling │
│ Last.fm API Key         │ Similar music candidate discovery     │
│ YouTube API Key / Token │ Playlist import & audio metadata      │
│ YouTube Playlist ID     │ Seed playlist for taste grounding     │
└─────────────────────────┴───────────────────────────────────────┘
```

---

### 1. Google Gemini API Key

Used for natural-language intent parsing, franchise candidate discovery, and budget-filler track curation.

1. Visit **Google AI Studio**: [https://aistudio.google.com/](https://aistudio.google.com/)
2. Sign in with your Google account.
3. Click the blue **"Get API key"** button in the top left navigation.
4. Select **"Create API key in new project"** (or choose an existing Google Cloud project).
5. Copy your generated key (starts with `AIzaSy...`).
6. Paste it into the **Gemini API Key** field in the prototype.

> **Note:** The prototype utilizes free-tier compatible Flash models and handles automatic retries and model fallbacks internally.

---

### 2. YouTube Data API Key or Bearer Token

Used to fetch video titles, channel names, and durations from your seed playlist. **No OAuth consent screen or localhost server is required** when using standard public or unlisted playlists.

#### Option A: Standard YouTube API Key (Recommended)
1. Go to the **Google Cloud Console**: [https://console.cloud.google.com/](https://console.cloud.google.com/)
2. Create a new project (e.g., `SiriusXM-Audio-Prototype`) or select an existing one.
3. In the search bar at the top, search for **"YouTube Data API v3"** and click **Enable**.
4. Go to **APIs & Services > Credentials**.
5. Click **+ CREATE CREDENTIALS** at the top of the screen and choose **API key**.
6. Copy the generated API key (starts with `AIzaSy...`).
7. Paste this into the **YouTube API Key or Bearer Token** field in the prototype.

#### Option B: Temporary OAuth Bearer Token (For Private Playlists)
If your playlist is strictly set to **Private**, you can generate a short-lived Bearer token:
1. Visit the **Google OAuth 2.0 Playground**: [https://developers.google.com/oauthplayground/](https://developers.google.com/oauthplayground/)
2. In the list of APIs on the left, scroll to **YouTube Data API v3** and select `https://www.googleapis.com/auth/youtube.readonly`.
3. Click **Authorize APIs** and sign in with your YouTube account.
4. Click **Exchange authorization code for tokens**.
5. Copy the `access_token` string (starts with `ya29...`) and paste it into the prototype.

---

### 3. Last.fm API Key

Used to query acoustic similarity graphs, artist top tracks, and album metadata based on the tracks found in your YouTube playlist.

1. Create or log in to a free account on **Last.fm**: [https://www.last.fm/](https://www.last.fm/)
2. Navigate to the **Last.fm API Account Creation Page**: [https://www.last.fm/api/account/create](https://www.last.fm/api/account/create)
3. Fill in the required fields:
   * **Contact Email**: Your email address.
   * **Application Name**: `SiriusXM Goal Mode Prototype`
   * **Application Description**: `Academic prototyping of audio recommendation systems.`
4. Leave **Callback URL** and **Homepage** blank.
5. Click **Submit**.
6. Copy your 32-character **API Key** (e.g., `b25b9595534c00b8fb5404374...`). You do not need the API secret.
7. Paste it into the **Last.fm API Key** field in the prototype.

---

### 4. Finding a YouTube Playlist ID or URL

The prototype accepts either a full YouTube playlist URL or a raw Playlist ID. The playlist must be set to **Public** or **Unlisted** (or use a Bearer token if private).

1. Open YouTube in your browser and open any music playlist.
2. Check the browser address bar. The URL will look like:
   ```text
   https://www.youtube.com/playlist?list=PL4fGSIqsQ875UvPxbP_gGkpxk_9t5vD8l
   ```
3. You can either:
   * Copy and paste the **entire URL** directly into the **YouTube Playlist ID or URL** field.
   * Or copy just the alphanumeric ID following `list=` (e.g., `PL4fGSIqsQ875UvPxbP_gGkpxk_9t5vD8l`).

---

## How to Run the Prototype

No server, Node.js environment, or package installation is required:

1. Download or locate `index.html`.
2. Double-click `index.html` or open it in any modern browser (**Google Chrome**, **Mozilla Firefox**, **Microsoft Edge**, or **Apple Safari**).
3. The interface will open with two tabs available:
   * **Interactive Prototype**: The live audio recommendation engine and queue generator.
   * **Project Overview & Strategy**: The strategic feature specification, Build/Buy/Partner evaluation, and team member details.

---

## Step-by-Step Usage Guide

### 1. Configure Credentials & Seed Catalog
1. Enter your **Gemini API Key**, **Last.fm API Key**, and **YouTube API Key / Token**.
2. Paste your **YouTube Playlist ID or URL**.
3. Click **"Load YouTube Song Catalog"**:
   * The app fetches videos from the playlist.
   * Gemini normalizes track titles and artist names.
   * Last.fm queries similar tracks to populate the **Candidate Recommendations Catalog**.
   * Progressive background resolvers fetch verified album artwork.

### 2. Formulate a Listening Goal
You can either click one of the **Preset Scenarios** (e.g., *Indie Discovery*, *Ambient Focus*, *Outside Discovery*) or formulate a custom goal:
* **Goal Description**: Freeform text (e.g., *"Recommend upbeat 80s synthpop and new wave for my morning run"*).
* **Duration Budget (Mins)**: Target total listening time (e.g., `30` minutes).
* **Explicit Exclusions**: Comma-separated topics, genres, or moods to reject (e.g., `ballads, live recordings, spoken word`).
* **Include Listener History Toggle**: Simulates a user with heavy conflicting past history (e.g., 80% serialized fiction) to verify that Goal Mode strictly obeys the current goal over habit.

### 3. Generate the Queue
Click **"Generate Goal Queue"**. The engine will:
1. Parse the prompt into a structured JSON payload (`normalized_goal`, `preferred_topics`, `excluded_topics`).
2. Evaluate candidates against exclusions and duration bounds.
3. Automatically perform **Prompt Expansion** or **Budget Filling** if candidates are insufficient.
4. Display the resulting queue with individual justifications, total duration metrics, compliance checks, and raw debug JSON.

---

## Key Architectural Mechanisms

```
 Listener Input ──────► Gemini Goal Parser ──────► Topic & Negative Filters
                                                          │
 Candidate Pool ◄────── Last.fm Similar API ◄─────────────┤
        │                                                 ▼
        ▼                                      Knapsack Duration Packing
 Prompt-Match Ingestion                                   │
 (External Discovery) ───────────────────────────► Budget Threshold Met?
                                                          │
                                                    No ───┴──► Budget Filler
                                                    Yes        Engine
                                                     │           │
                                                     ▼           ▼
                                             Final Validated Queue Display
```

1. **Deterministic Exclusion Filters**: Negative topic filtering happens in code before and after LLM generation, ensuring non-compliant tracks never reach the listener.
2. **Knapsack Duration Enforcement**: The engine tracks cumulative track seconds, preventing queue over-runs while ensuring at least 80% budget utilization.
3. **Multi-Model Dynamic Fallback**: In the event of Google AI Studio server load spikes, requests automatically migrate down the model chain without crashing the UI.
4. **Artwork Fallback Stack**:
   * Layer 1: High-resolution YouTube thumbnails (`maxres` / `standard` / `high`).
   * Layer 2: Last.fm `album.getInfo` and `track.getInfo` image structures.
   * Layer 3: Dynamic JSONP lookup via the Apple Music / iTunes Search API.

---

## Troubleshooting & FAQs

### Q: Why do I get a "403 Forbidden" or "API key not valid" error when loading YouTube?
* Make sure you have enabled the **YouTube Data API v3** in the Google Cloud project where your API key was generated.
* Verify that your API key does not have restrictive Application Restrictions (HTTP referrers) blocking local browser files (`file:///`).

### Q: The queue does not find enough songs for a very specific franchise prompt.
* Click the **"Load Songs for Prompt"** button situated above the Candidate Recommendations Catalog table. This instructs Gemini to query its music knowledge base and inject authentic tracks directly into the candidate pool.

### Q: Why does Last.fm say "Invalid API key"?
* Double check for leading or trailing whitespace when pasting the 32-character key from your Last.fm API account dashboard.

### Q: Is user data sent to external servers?
* No. Prompts, API keys, and history remain exclusively in browser memory and local session storage. Requests are made directly from your browser to Google, Last.fm, and YouTube endpoints.

---

## Course Submission Information
* **Project**: SiriusXM Pandora Goal-Based Audio Recommendation System
* **Deliverable**: Member C Interactive Prototype & Deliverables
* **Team**: Group 9
* **Members**: Hudson Trussell, Giang Khuu, Lira Sandoval Campos, Azimjon Izzatillaev
