---
layout: post
title: Get the last occurrences of in files with Bash
---

Yesterday I wanted to extract the last 2 reprojection errors from some
log files and came across/created a nice Bash one-liner to do the job:

```
for f in *_log.txt; do echo "$f" && grep "Reprojection error:" "$f" | tail -n 2; done
```

This will parse all your "_log.txt" files for the last 2 occurrences of the string "Reprojection error" and separate the results with a line of the filenames.

Some sample output:

```
1_log.txt
Reprojection error: 2
Reprojection error: 3
2_log.txt
Reprojection error: 2
Reprojection error: 3
3_log.txt
Reprojection error: 2
Reprojection error: 3
```
