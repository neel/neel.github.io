---
title: Listing Terms and Acronyms in Text or LaTeX using bash
date: 2020-12-26 15:35:35
tags: 
    - Latex
    - Linux
    - Utilities
categories:
    - Programming
---

I often need to check for inconsistent capitalization in my tex files. So listing all the consecutive capitalized words and characters helps me to decide which one is intentional capitalization and which one is not. The following bash script has two functions can lists all terms (Capitalized Phrase) and acronyms used throughout the input file.

```
$ terms filename.tex 
     19 Cloud Station
      9 Sensor Gateway
      7 Sensor Cloud Infrastructure
      ...
$ acronyms filename.tex      
     34 VM
     13 IaaS
     13 CPU
     ...
``` 

<!--more-->

To reuse save the code shown at the end as `$HOME/shortcuts.sh` then issue command `source $HOME/shortcuts.sh`. use `terms` and `acronyms` functions as shown below. And here is the `shortcuts.sh`.

```
#!/bin/bash
# source shortcuts.sh
# terms filename.tex
# acronyms filename.tex

terms(){
    grep -o -P "(?:[A-Z][a-z]+)\s+(?:\s*[A-Z][a-zA-Z]+)+" $1 | sort | uniq -c | sort -nr
}

acronyms(){
    grep -o -P "\b(?:[A-Z][a-z]*){2,}\b" $1 | sort | uniq -c | sort -nr
}
``` 
