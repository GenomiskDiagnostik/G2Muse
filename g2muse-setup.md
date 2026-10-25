# G2Muse — Installation & Setup (v2.3)

G2Muse shows messages on your Even Realities G2 glasses — and you can
reply by voice, right from the glasses.

**How it works (v2.3):** Everything goes through a **message relay**
(a secret GitHub Gist). Your Muse posts text messages to the relay;
the app polls the relay and shows them on your glasses. When you reply
by voice (hold-to-talk on the glasses), the app uploads the audio to
the relay, your Muse downloads and transcribes it locally, and replies
to what you said. No Telegram, no bots.

On your Muse's side, a **lightweight polling script** (no AI agent
involved) checks the relay every ~30 seconds and only wakes an agent
when you have actually spoken — so idle checks cost no tokens, and
replies typically land within about a minute of your voice message.

You need: a phone with the [Even Realities app](https://hub.evenrealities.com/docs),
your G2 glasses paired with the phone, and a (free) GitHub account.

## Step 1: Create the message relay (approx. 3 min, one time)

1. Go to [gist.github.com](https://gist.github.com) (log in with GitHub)
2. Create a **secret** gist: filename `g2muse-relay.json`, content
   `{"messages": []}` — click **"Create secret gist"** (not public)
3. Copy the **gist ID** from the URL (the long hex string after
   `gist.github.com/<username>/`)

## Step 2: Create a GitHub token (approx. 2 min, one time)

1. Go to [github.com/settings/tokens/new](https://github.com/settings/tokens/new)
2. Note: "G2Muse relay" — check the **`gist`** scope only
3. Generate and **copy the token** (you won't see it again)

**Security note:** the token gives access to all your gists (GitHub
can't scope it narrower). Keep it on your phone and with your Muse
only — never paste it in a chat. If it ever leaks or you lose your
phone, revoke it immediately at github.com/settings/tokens and create
a new one. Never put passwords or ID numbers in relay messages.

## Step 3: Connect the app to the relay

1. Open **G2Muse** in the Even app on your phone
2. Paste the **gist ID or gist URL** (from Step 1) and the **token**
   (from Step 2), tap **Save**
3. The app verifies the connection by reading the gist
4. Under **Settings** you can change:
   - **Auto-refresh** interval (default 60 sec)
   - **Language** — automatic from your phone's system language, or pick
     manually (14 languages)

## Using it on the glasses

The top line shows the **clock** (top right, updates every 30 sec).
Each message shows the **sender with date/time**, e.g.
`Muse · 18/10 02:18:`.

### Navigation (R1 ring)

- **Swipe up/down**: previous/next message (newest is no. 1)
- **Tap**: show the **full message** (tap again to go back)
- **Double-tap**: go **back** — from a message it jumps straight to the
  **idle screen** (handy in long lists: no scrolling all the way);
  double-tap again **on the idle screen** for the exit prompt
- **Hold** (long press): **voice reply** — hold while you talk, release
  to send (0.6 sec – 2 min, OGG/Opus)

### Idle screen

Navigation is circular with an **empty screen** between the last and
first message. The idle screen shows only the **unread count** (top
left) and the **clock** (top right) — no frame. When a new message
arrives while the idle screen is shown, it pops up briefly (8 sec.),
then the screen goes empty again.

### OS menu

- **Refresh** — fetch new messages manually
- **Delete message** — delete the currently shown message from the
  glasses (it stays in the relay)

### Read/unread

Each message header shows e.g. "3 unread" or "All read". The idle
screen shows only the unread count (nothing when all are read).

### Voice replies

Hold, talk, release. Your Muse transcribes the audio locally and
replies to what you said. The transcription is posted to the relay as
a message from you, so the glasses show the full conversation: your
words first, then the reply. **Safety rule:** before your Muse acts on a
voice message by deleting or overwriting anything, it will describe
what it heard and ask you to confirm with **YES!** (or reject with
**NO!**). Small things (questions, lookups) need no confirmation.

## Good to know

- 14 languages: Danish, Swedish, Norwegian, Greenlandic (Kalaallisut),
  Icelandic, Faroese, English, German, French, Russian, Ukrainian,
  Japanese, Chinese, Arabic
- Network: the app only contacts `https://api.github.com` (see
  `g2muse-2.0-netvaerkskontaktpunkter.md`)

---

# Appendix: Message to give your Muse

Copy the text below and give it to your Muse (in your main chat). It
instructs the Muse on setting up everything: security, relay,
polling, transcription, and the side chat.

---

> I want to use the G2Muse app (v2.3) with my Even Realities G2
> glasses. Please set up the complete pipeline for me:
>
> **1. Security rules (standing — think security into everything):**
> - My GitHub token gives access to ALL my gists. Store it ONLY in
>   your Secure Vault (never in chat, memory, files, or code).
> - Relay messages are unencrypted — never put passwords, ID numbers,
>   or other secrets in them.
> - If the token ever leaks or I lose my phone, revoke it immediately
>   at github.com/settings/tokens.
> - Before you DELETE anything or OVERWRITE existing content based on
>   a voice transcription, describe what you heard and get my
>   confirmation with JA! (or rejection with NEJ!). Transcription can
>   be misheard. Small things (questions, lookups, simple replies)
>   need no confirmation. This rule applies ONLY to voice, not text.
>
> **2. Message relay (GitHub Gist):**
> - I have created a SECRET gist; its ID is: `<PASTE GIST ID>`.
> - Ask me for the GitHub personal access token (gist scope) via the
>   Secure Vault flow and store it securely.
> - My messages to the glasses go through the relay: post JSON
>   `{"messages": [...]}` to the gist file `g2muse-relay.json` via the
>   GitHub API (keep the last 50 messages). Each message:
>   `{"id": "m_<timestamp>", "from": "Muse", "type": "text",
>   "text": "...", "ts": <unix seconds>}`.
>
> **3. Voice messages from the glasses:**
> - When I reply by voice, the app uploads the audio as base64 to the
>   gist as `voice-<timestamp>.ogg` and adds a `type: voice-note`
>   notification with an `audio_file` field pointing at it.
> - Set up local transcription with faster-whisper (small model, CPU)
>   so the audio never leaves your machine. Note: the app's
>   hand-muxed OGG/Opus is rejected by FFmpeg/PyAV — decode it by
>   manually demuxing the OGG pages and decoding Opus packets with
>   libopus (via ctypes), then transcribe the resulting WAV.
> - Post the transcription to the relay as a message from me
>   (`--from You`), BEFORE your reply — so my glasses show my
>   words as my message, then your reply underneath. The transcription
>   IS the receipt that you heard me: do NOT post a separate
>   "heard you" message or paraphrase of my words — just answer
>   what I said.
>
> **4. Polling / watch (token-cheap):**
> - Do NOT poll with a scheduled agent: an agent run every few minutes
>   burns tokens even when nothing happened. Instead set up a
>   lightweight event hook: a plain script (no LLM) that polls the
>   gist every ~30 seconds and only wakes an agent when a new
>   `type: voice-note` from me appears (track the watermark
>   `last_relay_voice_note` so each note is handled once).
> - The script must never see my real token: it fetches the gist
>   through the credential-surrogate mechanism (the runtime swaps
>   the surrogate for the real token on the way out; api.github.com
>   is the only allowed host). Nothing new = the script stays silent
>   and costs nothing.
> - **Claim lock (learned the hard way — one voice note got two
>   replies):** the watermark only advances when the woken agent has
>   FINISHED, and transcription can take longer than the poll
>   interval, so the next poll would see the same note as "new" and
>   wake a second agent. When the script wakes an agent for a note,
>   it must record a claim (the note's ts + the current time, in a
>   small state file) and refuse to wake again for that note while
>   the claim is fresh (~10 minutes). If the claim expires without
>   the watermark advancing, the agent is presumed dead and a new
>   wake is allowed.
> - The woken agent handles each new voice note (oldest first):
>   FIRST re-check the relay — if a transcription or a reply for
>   the note already exists, just advance the watermark and stop,
>   posting nothing (dedupe). Otherwise: download and transcribe
>   locally, post the transcription to the relay as my message,
>   reply to the content in my language, post the reply to the
>   relay as Muse, and update the watermark atomically.
>
> **5. Side chat & delivery:**
> - Create a dedicated side chat for the glasses (e.g. "Even Realities
>   G2"). Everything you write as a reply in that chat must ALSO be
>   posted to the relay (same text) so it lands on my glasses.
> - When I write to you from the side chat, relay this to the glasses,
>   and deliver your reply in both places: the side chat and the relay.
>
> Confirm each step as you complete it, and tell me when the whole
> pipeline is verified end-to-end (text both ways + one voice message
> transcribed correctly).

