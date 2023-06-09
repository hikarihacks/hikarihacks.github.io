---
layout: post
title:  "Writeup for Bsides Bucharest 2023 Web Challenge : nodebug"
tags: ctf web bsides 2023 
---


We are given a link to a webpage, let's see what we have there.
<img src="/assets/images/initial-webpage-nodebug.png" alt="Initial webpage for the challenge">

It expects a file parameter, let's assume it's called 'file' and let's try to read something that is 100% available to us : /etc/passwd

<img src="/assets/images/etc-passwd-nodebug.png" alt="Dump for the /etc/passwd file">

Alright, we can read files from the local system. Let's search for the source code and see what we can get.
Normally, we don't know the location of the source code on the file system, so we can try to dig for information using the information stored in the /proc/self directory.

Our main point of interest are /proc/self/environ and /proc/self/cmdline.

By dumping /proc/self/environ we find out the working directory of the application.
By dumping /proc/self/cmdline we find out the name of the source code file.

```js
const express = require('express');
const fs = require('fs');
const {
    cpuUsage
} = require('process');
const app = express(); // flag is in flag.txt 
app.get('/', (req, res) => {
    try {
        let file = req.query.file;
        let debug = req.query.debug;
        if (!file) {
            res.send("Missing file parameter");
            return;
        }
        if (debug) {
            file = new global[debug](file);
        }
        if (file.toString().includes('flag')) {
            res.send("Hacker detected");
            return;
        }
        res.send(fs.readFileSync(file, 'utf8').toString());
    } catch {
        res.send("Something bad happened");
    }
});
app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

Great, we found the source code!
Now, let's see what we can exploit here.

Normally, we would have to read the flag.txt file from the /usr/src/app folder (found using /proc/self/environ).
However. our file cannot contain the substring _flag_.

```js
 if (debug) { 
    file = new global[debug](file); 
 }
```

This code is the only place where our input is modified, so it's safe to assume that we have to force __file__ to become an object that has the following properties:
1. Can be converted to a string form
2. Could reference local files
3. The internal reference and the string representation should be different.

In order to exploit these conditions, the __URL__ class is exactly what we need.
We force __file__ to become an instance of an __URL__ with a _file://_ scheme. This way we can easily meet two out of three conditions.
In order to meet the last condition, we can use URL encoding that would internally be resolved to the normal file.

Therefore, our payload will be:
```bash
curl -s 'https://nodebug.fly.dev?debug=URL&file=file:///usr/src/app/fl%2561g.txt'
```

Please note that the string should be double URL encoded as we send it, because the first time it is automatically URL decoded by the backend. Practically, what the backend receives is actually 
```txt
file=file:///usr/src/app/fl%61g.txt
```

Thanks for reading and keep hacking :)
