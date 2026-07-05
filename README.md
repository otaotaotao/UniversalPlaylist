# UniPlay *(name still unsettled — see naming notes below)*

A "postcard" for playlists — hand someone your Spotify playlist without cluttering their Apple Music library.

Most playlist-conversion tools treat sharing across platforms as a permanent library sync. This is trying to be something smaller and more personal: closer to sending a postcard, or leaving a short intimate radio-diary aside on each track, than to migrating a whole collection.

**Live mock-up:** https://otaotaotao.github.io/UniversalPlaylist/
**Spec (English):** [uniplay_spec_en.md](./uniplay_spec_en.md)
**Spec (日本語):** [uniplay_spec.md](./uniplay_spec.md)

## Status

This is a personal, work-in-progress mockup, not a finished product. It's a static HTML file — no backend, no login, no account required on either end. Built with a lot of back-and-forth with Claude.

Known walls hit along the way (details in the spec): Spotify's Feb 2026 API changes killed unauthenticated playlist-track reads, Apple Music API access requires a paid $99/yr Developer account even for search, and matching accuracy on the free iTunes Search API isn't perfect (live versions/remixes sometimes resolve to the wrong recording).

## How it works, roughly

1. Export a Spotify playlist to CSV (e.g. via Exportify)
2. Embed the CSV into the HTML file, along with a title, a short message, and optional per-track comments and size weighting
3. Each track resolves to a direct Apple Music link via the free iTunes Search API
4. Deploy as a static page (this repo, via GitHub Pages)
5. Whoever you send it to taps through — no login, no sync, nothing added to their library unless they choose to

## Naming notes

The code and files are still under the placeholder name `UniPlay` / `Universal Playlists`. Current front-runners for an actual name: **Playlist Prayer**, **A Praylist for Us**. See the spec for the full list of candidates considered.

## License

MIT — see [LICENSE](./LICENSE). Idea's free; fork it, argue with the design choices, take it further.
