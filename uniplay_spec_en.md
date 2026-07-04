# UniPlay Spec v0.2 (name still unsettled)

## 0. Motivation & Philosophy

**Hand a Spotify playlist to someone using Apple Music, with as little friction as possible.** That's the baseline goal.

But this isn't meant to be just another format-conversion tool. What's actually in mind:

- **Music-sharing that feels like a diary entry.** Not a permanent collection that piles up in someone's library, but something that flows through, gets listened to, and passes on. It shouldn't clutter the recipient's Apple Music library.
- **A postcard-like feeling of sharing.** Not a cross-platform "sync" or "migration," but something as light and personal as sending a letter or a postcard.
- That's why the design language leans on stationery, wax seals, and a "P.S." tone — this isn't decoration, it's the philosophy expressed visually.

This philosophy should also be the yardstick for evaluating future features (auto playlist generation, other platform support, etc.). Any feature that's "convenient but makes things pile up permanently in the library" deserves a second look against this principle before it's added.

## 1. Where things stand: reality checked so far

### 1.1 Spotify side

- **On Feb 11, 2026, Spotify changed its Web API.** `GET /playlists/{id}/tracks` was replaced by `GET /playlists/{id}/items`, and critically, **track contents (items) are now only returned for playlists the authenticated user owns or collaborates on.** Under the Client Credentials Flow (no user login required), it's no longer possible to fetch track contents at all.
- Development Mode apps are now **capped at 5 authorized users** (changed Feb 7, 2026). Going beyond that requires applying for Extended Quota Mode, which demands things like registered-business status and 250k+ monthly active users — effectively unreachable for a personal hobby project.
- The app owner must personally hold a Spotify Premium subscription (a prerequisite for Development Mode).

→ Bottom line: **"just paste a Spotify link and auto-fetch" can't escape the 5-user cap even if you build your own OAuth login.**

### 1.2 Apple Music side

- The Apple Music API (MusicKit) **requires a signed token from a paid Apple Developer Program membership ($99/yr) for every operation, including search.** Not just writes — even read-only catalog search needs it.
- On the other hand, **there doesn't appear to be a Spotify-style "5 users max" cap.** Using MusicKit JS on the web, once you've paid the $99, it seems any number of end users can each log in with their own Apple ID and write to their own library.
- **Adding tracks to "Play Next" cannot be done even with the $99 paid tier.** That's a queue belonging to the native Apple Music app, entirely outside what an external website can touch. What MusicKit can control is its own embedded player — not the native queue on someone's iPhone.

### 1.3 Other technical notes

- The iTunes Search API (`itunes.apple.com/search`) is free and requires no auth, and can return a direct link to a song's top search hit (`trackViewUrl`). However, a plain `fetch` may be blocked by CORS, so the implementation tries `fetch` first and falls back to JSONP (as of July 2026, this hasn't been confirmed working on a real device yet).
- How this compares to existing tools:
  - **Soundiiz / TuneMyMusic / FreeYourMusic / SongShift** — full playlist conversion services. FreeYourMusic in particular prioritizes ISRC-based matching, which is close in spirit to what this project is going for.
  - **Odesli (song.link)** — only handles a single track or album at a time, not playlists.
  - **playlister.cc / Spotlistr** — convert a plain tracklist into both Spotify and Apple Music playlists. Testing showed good matching accuracy, but **track order got scrambled** — worth noting as a cautionary example, likely caused by parallel search requests that were never re-sorted back into original order before writing.
- Apple's own built-in "Transfer Music" feature (inside the Apple Music app) already lets someone save a playlist from Spotify and import it via Apple Music settings, for free, today. But it doesn't give you the "just send one link" experience.

## 2. Current architecture (mockup)

```
① Export a Spotify playlist to CSV via Exportify or similar (a version that scrolls
   through screenshots and OCR-extracts tracks via a Claude artifact has also been prototyped)
        ↓
② Embed the CSV directly into an HTML file (no database, no backend)
        ↓
③ For each track, query the iTunes Search API (title + artist + album) to get a
   direct link to the top hit. Falls back to a search-results page if resolution fails
        ↓
④ Deploy as a single static HTML file to GitHub Pages
        ↓
⑤ Recipient opens the page and taps a round icon button per track
   → Apple Music opens to that song's page → they add it to their own library/playlist manually
```

### 2.1 UI spec

- Stationery / message-in-a-bottle styled design (cream paper, dashed dividers, wax-seal-style checkmarks)
- Editable title and message field (currently placeholder text; will become free-form input once the editing feature ships)
- Per track: title / artist (own line) / album in 〈 〉 (own line) / round icon button (tapping opens the direct link and auto-checks the track; manual un-checking is also possible)
- Progress is not persisted (resets on reload). **Worth noting: this isn't a technical shortcut so much as a deliberate choice that happens to align well with the "doesn't pile up" philosophy in section 0.**

### 2.2 Data structure (current)

```json
{
  "title": "string",
  "artist": "string",
  "album": "string",
  "url": "string | null",   // resolved direct link from the iTunes Search API
  "resolved": "boolean",
  "done": "boolean"
}
```

ISRC/MBID aren't included yet since the CSV source data doesn't carry them (see section 4).

## 3. Architectures rejected or shelved

| Option | Status | Reason |
|---|---|---|
| Direct Spotify Web API access (Client Credentials) | **Rejected** | Feb 2026 API changes made fetching track contents impossible in principle |
| Spotify OAuth (Authorization Code Flow) | **Shelved** | Technically works (login flow was built). The 5-user cap is tolerable for personal use, but shelved due to the ongoing maintenance cost of keeping up with API changes |
| Auto playlist creation/writing via Apple MusicKit JS | **Shelved** | The $99 investment decision hasn't been made yet. No apparent user cap, so the cost/benefit case looks reasonably good — just not committed to |
| Auto-adding to "Play Next" | **Abandoned** | Confirmed to be impossible in principle from an external web app |

## 4. Roadmap

- [ ] Auto-generation from a pasted Spotify link (eliminating the manual CSV step in ①) — pending the "bring back OAuth" decision from section 3
- [ ] Font/design preset picker
- [ ] Editable title and message field (the current placeholder text was deliberately left as a preview of this feature)
- [ ] Plain-text export in a customizable template format
- [ ] Reverse version: Apple → Spotify (the same overall shape should largely carry over)
- [ ] YouTube Music / Amazon Music support (needs individual research into each platform's auth/API situation first)
- [ ] ISRC resolution via MusicBrainz (proper universal-ID support — already confirmed useful in practice for avoiding mismatches between live versions, remixes, and re-recordings)

## 5. Naming notes

- **Current implementation name: UniPlay** (footer reads "Universal Playlists"). This is a placeholder, not finalized.
- **Naming is still unsettled.** Current front-runners:
  - **Playlist Prayer**
  - **A Praylist for Us**
  - (other candidates considered along the way: Praylist on its own, universal music drifter、tracks across the universe  etc. — the direction has generally leaned toward either the pray/play double meaning, or the drift/tide "doesn't accumulate" idea)
- Code and file names will stay as `uniplay` for now, with a bulk rename once a name is settled.
- Design motifs: stationery, message in a bottle, wax seal
- Tone of voice: letter-like lightness, in the spirit of a "P.S."
