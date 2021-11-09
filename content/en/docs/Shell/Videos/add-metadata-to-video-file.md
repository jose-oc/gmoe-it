---
title: "Add metadata to video file"
date: 2021-11-09T17:55:31+01:00
draft: true
---

Add metadata to video file using ffmpeg.

```sh
ffmpeg -i input.mp4 -metadata title="This is my metadata title for the video" \
-metadata comment="'This is my metadata title in the comment for the video'
(by me)
---
Add the comments you want to the video file so it is kept within the video file.
---
1. Step 1
2. Step 2
---
All done." \
-c copy output.mp4
```