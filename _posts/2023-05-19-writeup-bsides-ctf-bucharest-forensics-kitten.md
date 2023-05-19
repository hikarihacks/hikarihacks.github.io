---
layout: post
title:  "Writeup for Bsides Bucharest 2023 Forensics Challenge : Kitten"
tags: ctf forensics bsides 2023 
---

I feel like this challenge would be more of a misc challenge than a forensics one, but who am I to argue with the organizers?

You can download the challenge file ![here](/assets/files/kitten).


I was hoping to find an easy win using the __file__ command, however it didn't help that much.

<img src="/assets/images/file-kitten.png" alt="Screenshot containing the output of the file command on the input file">

Still, everything about the challenge offers hints if you know the answer :)

The file format of this file is SIXEL. It represents bitmap graphics using ASCII characters. However, the catch is that not every terminal supports SIXEL graphics.

![Here](https://www.arewesixelyet.com) is a list of terminals that support the SIXEL graphics format.

I was using xfce-terminal during the CTF, so no luck for me :)

However, if you are using a Linux distribution with a KDE desktop environment, you could solve the challenge simply by using 
```bash
cat kitten #can you spot the obvious hint dropped by the challenge author? :)
```

This is the output in a terminal with SIXEL support:

<img src="/assets/images/kitten.png" alt="Screenshot containing the flag">

Keep hacking :)
