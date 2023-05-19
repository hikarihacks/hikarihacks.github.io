---
layout: post
title:  "Writeup for Bsides Bucharest 2023 Misc Challenge : Unzip Me"
tags: misc ctf bsides 2023
---

We are <a href="/assets/files/l0amyydk1p.zip">given</a> a password protected ZIP archive with a weird name.

My first idea was to try and crack it using John The Ripper, so I created a hash using

```bash
zip2john l0amyydk1p.zip > hash
```

Then, I tried cracking it using the rockyou.txt wordlist

```bash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Still, the cracking attempts failed.
Then, I tried to unzip it manually and took a closer look at the seemingly random filenames. They followed the same pattern, so I would expect both of them to be password encrypted. What password could be easily discoverable and automatable in order to create the challenge? 

My first answer to this question proved to be the right one :)
The password for each archive turned out to be its' name.
I quickly wrote a Python script to automate the archive extraction and cleanup.

```py
#!/usr/bin/env python3
from zipfile import ZipFile
from os import remove
def extract_once(zip_filename : str):
    password = zip_filename.split(".")[0].encode()
    with ZipFile(zip_filename) as f:
        _next = f.namelist()[0]
        print(f"Next : {_next}")
        f.extractall(pwd=password)
    return _next 

if __name__=="__main__":
    initial_name = 'l0amyydk1p.zip'
    try:
        while True:
            _next = extract_once(initial_name)
            remove(initial_name)
            initial_name = _next
    except:
        with open('flag') as f:
            print(f.read())
```

Enjoy the flag and keep hacking :)
