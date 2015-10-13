---
layout: post
title: bash script/knowledge collection
---

This is a collection of different bash scripts or facts I might need sometime.

Find your local IP: `ip route get 8.8.8.8 | head -1 | cut -d' ' -f8` [Source](http://stackoverflow.com/a/25851186/497116)  
Size of subfolders: `ls -AF | grep \/ | sed 's/\ /\\\ /g' | xargs du -sh` [Source](http://www.toomanyredirects.com/listing-all-subdirectories-with-file-sizes-in-linux/)  
