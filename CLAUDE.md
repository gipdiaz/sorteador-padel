# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page web application called "Sorteador de Parejas de Padel" (Padel Pair Randomizer). The application is a standalone HTML file with embedded JavaScript.

## Project Structure

```
.
└── index.html          # Main application (HTML + embedded JavaScript)
```

## How to Run

Simply open `index.html` in any modern web browser. No build process, compilation, or web server required.

## Application Architecture

The application is entirely self-contained in [index.html](index.html):

- **Frontend**: Uses Bootstrap 5.3.0 for styling (loaded via CDN)
- **Backend**: Firebase Realtime Database and Firebase Authentication (via CDN)
- **Authentication**: Google OAuth using Firebase Auth (signInWithPopup method)
- **Logic**: Vanilla JavaScript embedded in the HTML file
- **State Management**: Firebase Realtime Database stores user data, players, and draw history

### Key Functionality

1. **Google Authentication**: Users must sign in with Google OAuth to create draws (lines 149-159)
2. **Player Input**: Collects 4 drive players and 4 backhand players via text inputs with autocomplete (lines 40-51)
3. **Player Autocomplete**: Suggests previously used players from Firebase (lines 214-222)
4. **Pair Generation**: Randomly pairs one drive player with one backhand player (lines 299-303)
5. **Matchup Creation**: Randomly creates 2 matchups from the 4 pairs (lines 315-323)
6. **Duplicate Detection**: Tracks previous draws in Firebase and warns if the same player combination is attempted again within 24 hours (lines 269-279)
7. **Share Draws**: Generate unique shareable links for draws (lines 360-394)
8. **View Shared Draws**: Anyone can view draws via shared links without authentication (lines 419-478)
9. **User Profile**: Displays user's Google photo and name in header (lines 172-177)

### Important Code Sections

- **Firebase Configuration**: lines 110-125 (MUST be configured with real Firebase credentials)
- **Authentication Management**: lines 133-189
- **User Data Loading**: lines 191-222
- **Main sorting logic**: lines 234-345
- **Player autocomplete**: lines 347-356
- **Share functionality**: lines 358-415
- **Shared draw view**: lines 417-478
- **Warning message**: line 271

### Firebase Data Structure

```
users/
  {userId}/
    ├── profile/           # User's Google profile info
    ├── players/           # Array of unique player names for autocomplete
    └── historialDia/      # 24-hour history of draws
        └── {sorteoKey}/
            ├── count      # Number of times this combo was attempted
            └── timestamp  # When it was created

sorteos/                   # Public shared draws
  {sorteoId}/
    ├── owner              # User ID of creator
    ├── ownerName          # Display name of creator
    ├── jugadoresDrive     # Array of drive players
    ├── jugadoresReves     # Array of backhand players
    ├── parejas            # Array of generated pairs
    ├── enfrentamientos    # Array of matchups
    └── created            # Timestamp
```

## Setup Instructions

### Firebase Configuration (Required)

Before the app works, you MUST create a Firebase project and update the configuration:

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create a new project (or use existing one)
3. Enable **Google Authentication**:
   - Go to Authentication → Sign-in method → Enable Google
4. Create a **Realtime Database**:
   - Go to Realtime Database → Create Database
   - Start in **test mode** (or configure security rules later)
5. Add authorized domain:
   - Go to Authentication → Settings → Authorized domains
   - Add: `gipdiaz.github.io`
6. Get your Firebase config:
   - Go to Project Settings → Your apps → Web app
   - Copy the firebaseConfig object
7. Replace placeholder config in [index.html:112-120](index.html) with your real credentials

### GitHub Pages Deployment

- Deployed at: https://gipdiaz.github.io/sorteador-padel/
- Uses `signInWithPopup()` for OAuth (works on static hosting)
- HTTPS is automatically provided by GitHub Pages

## Development Notes

- All JavaScript is embedded directly in the HTML file (no separate JS files)
- No CSS files - styling is handled entirely by Bootstrap CDN
- No build tools, bundlers, or package managers needed
- Uses Firebase for backend (authentication + database)
- The warning message on line 271 contains colorful Spanish language that may need adjustment for different audiences
- Shared draws can be viewed without authentication
- User must be authenticated to create new draws
