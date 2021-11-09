---
title: "Add metadata to video file"
date: 2021-11-09T18:55:31+01:00
draft: true
---

This is a little bash script to extract the audio from a video in the m4a format to be played back in a podcast app.

```sh
for FILE in *.mp4;
do
    echo -e "Extracting audio from '\e[32m$FILE\e[0m'";
    ffmpeg -i "${FILE}" -vn -c:a copy "${FILE%.mp4}.m4a";
done
```
