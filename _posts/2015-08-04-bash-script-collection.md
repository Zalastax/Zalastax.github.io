---
layout: post
title: bash script/knowledge collection
---

This is a collection of different bash scripts or facts I might need sometime.

Find your local IP: `ip route get 8.8.8.8 | head -1 | cut -d' ' -f8` [Source](http://stackoverflow.com/a/25851186/497116)  
Size of subfolders: `ls -AF | grep \/ | sed 's/\ /\\\ /g' | xargs du -sh` [Source](http://www.toomanyredirects.com/listing-all-subdirectories-with-file-sizes-in-linux/)  

Kill a users long running MySQL queries: `echo "SHOW FULL PROCESSLIST;" | mysql -uroot -pROOTPASSWORD -h localhost | sed '1d' | cut -f 1,2,6 | awk '{ if ($3 > 5 && $2 == "usertokill") print "KILL " $1}' | mysql -h localhost -u usertokill -pPASSWORD`
