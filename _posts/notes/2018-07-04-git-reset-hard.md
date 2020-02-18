---
title: "Recovery from `git reset --hard`"
date: 2018-07-04
permalink: /notes/2018/07/04/git-reset-hard
--- 

We've all been there. Frustrated at the mess we've created locally. Impulsively, we type it in:

```bash
$ git reset --hard
```

Now, we run a quick `git status` and say to ourselves "oh fuck. oh fuck. I really fucked up didn't I?".
This happened to me pretty recently and I lost a whole bunch of blog posts that I really didn't want to remember/rewrite. So how does one recover from a hard reset?

First of all, note that only files that have been `git add`ed can be reacquired. They are stored in the `.git/lost-found/other` folder as blobs. We're going to use the command `git fsck --lost-found` command to look at all the dangling blobs, loop through each blob, `git show` each blob, and search for the string in the contents. Here is the full implementation written in python3:

```python
import os

all_blobs = os.popen('git fsck --lost-found').read()

string_to_find = 'Logistic Regression'

for l in all_blobs.splitlines():
blob = l.split()[2]
try:
contents = os.popen(f'git show {blob}').read()
if string_to_find in contents:
print(f'{string_to_find} found in {blob}:')
print(contents)
except:
pass
```

In the example above, I lost my machine learning notes on Logistic Regression and ran this. Voila! I found the blob and the contents of my blog post.

Hopefully this was helpful. Happy finding and `fsck`ing!
