# Fencive Vanguard Assistant — Project Documentation

## Overview
Frontend chatbot UI for Fencive Vanguard alarm system support. Clean, accessible design with integrated voice mode (push-to-talk). Hosted on GitHub Pages.

**Live:** https://ghassane91.github.io/fencive-vanguard-assistant/  
**Repo:** https://github.com/Ghassane91/fencive-vanguard-assistant

---

## Architecture

### Tech Stack
- **HTML/CSS/Vanilla JS** — no frameworks, single-file design
- **Audio API** — Web Audio Context + MediaRecorder
- **Hosting** — GitHub Pages (static)
- **Backend** — n8n webhooks (not in this repo)

### Layout
- `position: fixed; inset: 0` on `#app` (iOS Safari fix)
- Flexbox column: header → messages → input bar
- Messages: `flex: 1; min-height: 0; overflow-y: scroll; -webkit-overflow-scrolling: touch`
- Responsive: max-width 860px, centered

### Color Scheme (Fencive Brand)
```css
--bg:         #ffffff;     /* page background */
--surface:    #ffffff;     /* cards, surfaces */
--surface2:   #f5f5f5;     /* hover, secondary */
--border:     #e8e8e8;     /* borders, dividers */
--primary:    #f0f0f0;     /* buttons */
--primary-hover: #e5e5e5;
--primary-text: #666666;   /* button text */
--text:       rgba(0,0,0,0.8);
--muted:      rgba(0,0,0,0.5);
--muted2:     rgba(0,0,0,0.3);
```
All white/gray palette — no black, no colors.

---

## Features

### Chat
- Welcome screen with 5 quick suggestions
- User/bot bubble layout with language badges (FR/AR auto-detect)
- Smart scroll: only auto-scrolls if near bottom (threshold 120px)
- Typing indicator animation
- Session ID for conversation tracking

### Voice Mode (Push-to-Talk)
**Interaction:** Click "Mode vocal" button → overlay opens → hold sphere to record → release to send

**State Machine:**
```
idle → (hold) → listening → (release) → thinking → speaking → idle
```

**Audio Processing:**
- `getUserMedia()` called synchronously inside tap handler (iOS requirement)
- MediaRecorder with auto-detected codec: `audio/webm` or `audio/mp4`
- Downsample to 16kHz WAV format
- Send to `/webhook/voice-bot-fencive` endpoint

**iOS Quirks Fixed:**
- AudioContext must resume inside synchronous user gesture (tap)
- MediaRecorder on iOS only supports `audio/mp4`, not `audio/webm`
- `overflow: hidden` on body breaks iOS touch event propagation
- Use `position: fixed; inset: 0` instead of `height: 100dvh + overflow: hidden`
- `-webkit-overflow-scrolling: touch` for native momentum scroll
- `touch-action: manipulation` + `-webkit-touch-callout: none` for orb

**Backend Integration:**
- Chat: `POST /webhook/chat-bot-fencive/chat` → `{chatInput, sessionId}`
- Voice: `POST /webhook/voice-bot-fencive` (audio/wav binary) → `{transcription, reponse, audio_base64}`
- Response audio plays through speaker with waveform visualization

---

## Voice Sphere UI
- **Black circle, 90px diameter**
- Centered in overlay with backdrop
- **Orb states:**
  - `.listening`: darker (#333), scale 0.96, animated pulse ring
  - `.speaking`: lighter (#555)
  - `.thinking`: pulsing opacity animation (throb)
- Pulse ring animation (scale 0.9 → 1.7, opacity 0.4 → 0)
- SVG microphone icon inside

---

## File Structure
```
index.html           — single file, all code
.claude/launch.json  — preview server config
```

**No build step.** Pure HTML/CSS/JS.

---

## API Endpoints
```
CHAT:  http://localhost:5678/webhook/chat-bot-fencive/chat
VOICE: http://localhost:5678/webhook/voice-bot-fencive
LOGO:  https://fencive.com/cdn/shop/files/logo_base_f31c08b0-884c-460e-834c-ac98632b88f6.png?height=60
```

Update these in the `<script>` section if backend URL changes.

---

## Development

### Run Locally
```bash
npm install -g serve
serve -s . -l 3000
# Visit http://localhost:3000
```

### Deploy to GitHub Pages
Changes to `master` auto-deploy via GH Pages.

```bash
git add index.html
git commit -m "..."
git push origin master
```

### Cache Issues
GitHub Pages aggressively caches. Users may need:
- Hard refresh: `Cmd+Shift+R` (Mac) / `Ctrl+Shift+R` (Windows)
- Or: Settings → Safari → Clear History and Website Data

---

## Known Issues & Solutions

### Voice mode doesn't work on iOS Safari
**Root cause:** `getUserMedia` not called synchronously in gesture handler, or AudioContext not resumed inside tap.
**Solution:** Ensure `orbTap()` or `startRec()` is called directly from `touchstart`/`click`, no async chains before.

### Scroll broken on iOS
**Root cause:** `overflow: hidden` on body after scroll breaks touch propagation.
**Solution:** Use `position: fixed; inset: 0` on app container instead.

### Double-tap triggers recording twice
**Root cause:** Both `touchstart` and `click` fire on mobile, calling handlers twice.
**Solution:** Debounce or check state before proceeding.

### Message doesn't scroll into view on iOS
**Root cause:** `scrollTop = scrollHeight` doesn't work reliably on iOS flexbox.
**Solution:** Use smart scroll threshold (only scroll if user is near bottom, within 120px).

---

## Testing Checklist

- [ ] Chat works: send message, get response
- [ ] Suggestions are clickable, fill input
- [ ] Voice button opens overlay
- [ ] Orb holds down → "Je vous écoute…"
- [ ] Release → "Je réfléchis…" → plays response
- [ ] Can scroll conversation while voice mode open
- [ ] Language detection (FR/AR badges)
- [ ] Typing indicator shows
- [ ] Colors are white/gray (no black/blue)
- [ ] Responsive: mobile, tablet, desktop

---

## User Preferences
- **Language:** French (français)
- **Style:** Sobre, accessible, tech-touch
- **Colors:** Fencive brand — white/gray only
- **Voice:** Push-to-talk (hold → speak → release → send)
- **Mobile-first:** iOS Safari priority

---

## Session Context
**Last updated:** 2026-06-25  
**Current state:** White/gray theme, push-to-talk voice mode, chat with suggestions, GitHub Pages hosting  
**Next:** Test on iPhone, verify voice works, iterate on UX

---

## Useful Git Commands
```bash
# See recent commits
git log --oneline -10

# Reset to previous version (if needed)
git reset --hard <commit-hash>
git push origin master --force

# Check deployment
git log --oneline | head -3
```

---

**Questions? Check commit messages or this file.**
