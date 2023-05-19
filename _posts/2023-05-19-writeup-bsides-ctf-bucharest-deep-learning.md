---
layout: post
title:  "Writeup for Bsides Bucharest 2023 Misc Challenge : Deep learning"
tags: misc ctf bsides 2023
---

I have to admit that I was really scared to give this challenge a try, because I don't have a lot of experience with AI in general.

We are <a href="/assets/files/model.pth">given</a> file with a .pth extension.
I had no idea what a .pth extension is and how to interpret the data there, so after a bit of <a href="https://letmegooglethat.com/?q=what+is+a+pth+file">googling</a>, I found out that the .pth file is used by PyTorch, a Python machine learning library.

I loaded it in Python and got stuck. What activation function do I use? How did the author want us to discover the flag if we only have the weights? 

The <a href="https://stackoverflow.com/questions/67657926/how-to-load-pth-file">answer</a> at the end of the file helped me a lot and directed my focus on the weights, which led me to an interesting observation.
I printed the model representation in Python and it seemed to me that the values in the tensor are ASCII.
WE ONLY HAVE THE WEIGHTS. This was the solution.

Here is the script that prints the flag:
```py
#!/usr/bin/env python3
import torch

weights = torch.load('model.pth')
a = weights['layer.weight'].detach().cpu().numpy()
result = ""
for char in a[0]:
    result += chr(int(char))
print(result)
```

Thanks for reading and happy hacking :)
