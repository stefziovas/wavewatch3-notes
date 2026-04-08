# Miscellaneous Useful Commands

## 1) Search command history

history | grep curl

## 2) Search inside files

#searching for key word: OUTS in all files
grep -r OUTS *

#for only searching in switch files
grep "OUTS" switch_*

#If you aren't sure if it's "OUTS", "outs", or "Outs", add the -i flag:
grep -i "OUTS" switch_*

#If you only need a list of the files that contain the word, add the -l flag:
grep -l "OUTS" switch_*

## 3) Check file size

du -sh wind.ww3 
