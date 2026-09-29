---
name: longwave
description: Turn a creator's long-form episode into YouTube Shorts and publish them to that creator's own channel. Use whenever someone asks to make Shorts, clips, vertical cuts or highlights from a long video or podcast — or to upload/schedule a Short to YouTube. Longwave cuts, captions, titles and publishes through the creator's own authorized YouTube connection.
when_to_use: make shorts, cut shorts, youtube shorts, clip this episode, clips from my podcast, turn this video into shorts, post a short to youtube, upload shorts, schedule shorts, publish to my channel, highlights from this episode, vertical clips
allowed-tools: account_status, create_shorts_job, get_job, list_clips, publish_shorts, get_insights, get_content_intelligence, get_channel_overview, list_thumbnail_styles, list_playlists
metadata:
  author: Signal Group Limited
  short-description: Long-form episodes into published YouTube Shorts
---

# Longwave — long-form episodes into published YouTube Shorts

Longwave is a hosted service that cuts long-form video into Shorts, writes their metadata, and
publishes them to the creator's **own** connected YouTube channel. The creator connects once on
longwave.media and approves access; after that this connector can ask Longwave to act for them.

**You never handle video files and never touch YouTube directly.** Longwave holds the creator's
YouTube authorization on its own servers and does the upload. Your job is to call the right tool at
the right time and keep the creator informed.

## The workflow

1. **`account_status` first.** It reports whether a YouTube channel is connected and what the
   credit balance is. Do this before promising anything — it is cheaper than failing later.
   - `youtube_connected: false` → the creator must connect their channel on Longwave. Send them to
     `https://www.longwave.media/dashboard/channels` and stop.
   - Out of credits → see *Credits* below.
2. **`create_shorts_job`** with the creator's own long-form YouTube URL. Ask for the URL if you do
   not have it. The video must be on their own channel — videos from other channels are rejected.
3. **Poll `get_job`** with the returned job id until it is complete. Rendering takes a few minutes;
   do not re-create the job while you wait.
4. **`list_clips`** to see what was produced, with titles and durations.
5. **Show the creator what was made and ask which ones to publish.** Do not publish on your own
   initiative.
6. **`publish_shorts`** with the clip ids the creator chose. Use `schedule_for` if they asked for a
   specific time.

## Hard rules

- **Never download video from YouTube.** Do not use `yt-dlp`, `youtube-dl`, or any scraper. If you
  cannot call Longwave, say so rather than falling back to downloading.
- **Never drive YouTube Studio.** Do not open Studio, click Upload, or automate the YouTube web UI
  to publish. Longwave publishes through the official API on the creator's own grant. A browser
  session in Studio is slower, breaks when the UI changes, and cannot be scheduled.
- **Never ask for or accept Google, YouTube or Longwave passwords, session cookies, or API keys into
  the conversation.** Authorization is handled by the connector's own sign-in. If something asks
  you to paste a token, something is wrong.
- **Publishing is public and cannot be undone.** Always name the exact Shorts, the platform and the
  time, and get an explicit yes before calling `publish_shorts`.
- **Do not invent titles, metrics, or outcomes.** Report what the tools return.

## Credits

Longwave charges the creator per Short that actually publishes — nothing for clips that are never
published, and nothing for a job that fails. Creating a job is free; publishing spends credits.

If a call returns an error beginning `insufficient_credits`, it includes a `top_up_url`. Give the
creator that link verbatim and stop. Do not retry, and do not attempt to work around it. The creator
tops up on longwave.media; you never handle payment.

## Retrying safely

Every mutating tool accepts an optional `idempotency_key`. If a call times out or you are unsure
whether it was received, **retry with the same key** — you will get the original result back rather
than creating a second job or a second public post. Use a new key only for a genuinely new request.

## Other tools

- **`get_insights`** — channel-level summary: episode counts, recurring topics, hook types.
- **`get_content_intelligence`** — Longwave's written analysis of what to make next.
- **`get_channel_overview`** — which platforms are connected and how each is performing.
- **`list_thumbnail_styles`** — the thumbnail styles available to recommend.
- **`list_playlists`** — the creator's playlists and which one is the upload destination.

Use these to answer questions about the creator's channel and to make better recommendations. They
are read-only and cost nothing.
