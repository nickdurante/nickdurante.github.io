---
title: "Fix Jellyfin Subtitles"
categories:
  - self-host
comments: true

---

Linked to this [Github Issue](https://github.com/jellyfin/jellyfin/issues/12113)

I've encountered many problems in Jellyfin streaming to a Chromecast Gen 2.

Subtitles from different languages and different timestamps would merge and overlap on the screen.

Modifying these default parameters seems to solve the problem:

* Admin panel -> Playback -> Transcoding -> Uncheck: Allow subtitle extraction on the fly

* Install plugin Subtitle Extract (v4.0.0.0) -> plugin settings -> Check: Extract subtitles during library scan

