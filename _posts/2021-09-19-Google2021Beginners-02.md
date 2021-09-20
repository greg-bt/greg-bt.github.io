---
layout: writeup
title: "Challenge #2 - Moscow Apartment"
ctf: Google CTF 2021 - Beginners Quest
nextWrite: "Google2021Beginners-03"
---
For this challenge we are given a file.
Running the file command we can see the file is a Zip archive. So opening the file using 7zip we find `logic-lock.png` which contains a logic circuit which will reveal the flag when it is solved.

![logic]({{ site.baseurl }}/images/logic-lock.png)

Back tracking the circuit we find the correct combination is: BCFIJ

Therefore the flag is `CTF{BCFIJ}`

