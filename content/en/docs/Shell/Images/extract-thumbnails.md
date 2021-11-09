---
title: "Extract thumbnails"
date: 2021-11-09T17:50:31+01:00
draft: true
---

## Command to extract thumbnails from my RAW photos 

so I can have all of them in my hard disk in a reduced space.
When I want to process a RAW I can look for the photo using the thumbnails, then get the original RAW from my external storage.

Requirement: having `dcraw` installed.

Directory where I move RAW images after extracting its thumbnail. Backup just in case.

```shell
mkdir -p ~/tmp/raw-photos-to-be-deleted/
find ./raw-photos-directory -type f \( -name "*.nef" -or -name "*.arw" \) -exec dcraw -e "{}" \; -exec mv "{}" ~/tmp/raw-photos-to-be-deleted/ \;
 ```
