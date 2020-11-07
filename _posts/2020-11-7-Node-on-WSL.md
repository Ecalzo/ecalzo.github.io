---
layout: post
title: Nodejs and npm setup on WSL
---

Recently, I've been focused on developing a Chrome extension, [Skater](https://github.com/Ecalzo/Skater). This extenion started off as a hackathon project among friends, resulting in a scrappy, messy codebase written in vanilla js. While a lot of fun to develop at the time, revisiting and making changes without a testing framework in place has been a headache. I've made the decision to revisit the extenion and implement a testing suite with [jest](https://jestjs.io/). You can follow along with those updates on the [jest-implement branch](https://github.com/Ecalzo/Skater/tree/jest-implement). Being an occasional Windows Subsystem for Linux (WSL) user, I wanted to get node and npm set up properly, so that I can switch between Mac and PC at will. 

# Dos and Don'ts
Don't:
* Install Node via a Windows installer (if you plan on using it with WSL)
* Don't even `apt-get install nodejs`

Do:
* Install [nvm](https://github.com/nvm-sh/nvm)

# Why?
If you have used the `apt-get` command to install nodejs, you might get an error like this for npm:

```
: not foundram Files/nodejs/npm: 3: /mnt/c/Program Files/nodejs/npm:
: not foundram Files/nodejs/npm: 5: /mnt/c/Program Files/nodejs/npm:
/mnt/c/Program Files/nodejs/npm: 6: /mnt/c/Program Files/nodejs/npm: Syntax error: word unexpected (expecting "in")
```

This is a known issue, and there's an [entire GitHub thread about it](https://github.com/Microsoft/WSL/issues/1512). 


As someone who spent way too much time trying to fix this, I encourage you to go the nvm route for setting up a dev environment on WSL.
