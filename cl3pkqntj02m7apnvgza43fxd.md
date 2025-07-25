---
title: "5 Things I Learned Trying To Publish A Simple NPM Package"
datePublished: Sat May 28 2022 07:51:23 GMT+0000 (Coordinated Universal Time)
cuid: cl3pkqntj02m7apnvgza43fxd
slug: 5-things-i-learned-trying-to-publish-a-simple-npm-package
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/IPx7J1n_xUc/upload/v1653719447465/Asl_6Zoco.jpeg
tags: javascript, nodejs, npm, package

---

## Preface

While this is not my first published NPM package, I learned some cool tips that took me way too long to find/troubleshoot this time around so I thought I’d share them with you. I’d also like to preface this post by saying I am a Windows user, so some situations may be non-existent for Mac/Linux users.

## Introduction

As I mentioned above, this is not my first package. I did make a package called [fast-form-validator](https://www.npmjs.com/package/fast-form-validator) as a way to streamline form validation for simple websites. This time, however, I came across a YouTube video discussing linting your git commits using Husky for easier implementation of git hooks, in my case a pre-commit hook and Commitlint to ensure that my commits are in line with [conventional commits' Standards](https://www.conventionalcommits.org). The process of setting up the two modules was not long at all. I was simply curious If I could implement a single module that would make the set-up process faster and easier just for the challenge. I had seen this done before by other developers and I simply wanted to try it for myself.

## Steps

### 1: Windows users install git for Windows and take advantage of bash scripting!

---

Git for Windows provides a BASH emulation used to run Git from the command line. This is great since a lot of times we are given instructions intended for Mac users that won't work on Windows machines. This was a real pain to pin down because I thought my scripts were running with git bash out of the box but kept getting error messages akin to Command Prompt instead. So, ensure to set the shell you want to be used when you call NPM:

```plaintext
npm config set script-shell "C:\\Program Files (x86)\\git\\bin\\bash.exe"
```

or (64bit installation)

```plaintext
npm config set script-shell "C:\\Program Files\\git\\bin\\bash.exe"
```

### 2: You don’t have to publish your package to NPM to test if your package works!

---

Of course, there had to be an easier way to test your package locally, with the first package I was so eager to see working code that I didn't even stop or slow down to discover the simpler way I know now.

NPM and NPX can both be executed on local repositories!

%[https://giphy.com/gifs/5VKbvrjxpVJCM] 

```plaintext
npm install 'path/to/repo'
```

and to simulate an npm package, you can run:

```plaintext
npm pack
```

This will create the gzipped tar file (your-package-name-versionNumber.tgz) for you to reference.

So in the end it should look something like this:

```plaintext
npm install 'path/to/my-package/my-package-1.0.0.tgz
```

### 3:NPX cache on Windows does not clear properly!

---

Whew! This one was a doozie to pin down. If you are implementing npx in your package, updates won't show on successive installs and you would need to clear the cache each time. Thankfully at some point, I noticed the error messages had references to code I was sure I had changed and I figured it was not clearing with the usual `npm cache clean –force` and in some cases, the files that are located on Windows at `%LocalAppData%/npm-cache/_npx` were locked and didn't allow me to delete them manually. Ironically, there is a package that will delete it for you:

```plaintext
npx clear-npx-cache
```

### 4:NPM won't include all your files unless you tell it to!

---

Part of my module involves using a config file but I thought it would be bundled automatically with `npm pack` however, errors revealed it did not exist in my archive, I finally found out that there is a files property you can add to your package.json that explicitly tells npm what to include when you run `npm pack` here's a look at mine:

```json
"bin": "./bin/index.js",
  "files": [
    "bin",
    ".husky",
    ".commitlintrc.json" <-- config file that was left out
  ],
```

### 5:Create a test repo with preloaded scripts to make life easier

This time around I made sure to work smart and not hard so I created a test repo (mine is called test-dummy) to install my package and ensure it behaved as intended. Since I would constantly have to reset the test environment on each update, I quickly realized I could use npm scripts in the dummy's package.json to simplify my workflow.

These scripts:

* reset the environment by undoing the changes (installation of my package) using git.
    
* Remove the node\_modules folder
    
* Clear the npx cache folder
    
* Clear the npm cache folder
    
* Show me that the npm cache is indeed clean (npm cache verify)
    
* Execute my package via npx
    
* Installed my package via npm
    
* Print a message to the screen that the environment was reset
    

This is how it looks:

```json
"scripts": {
    "install-package": "npm install -D --foreground-scripts ../commitlint-with-husky/commitlint-with-husky-2.0.0.tgz",
    "execute-package": "npx --yes ../commitlint-with-husky/commitlint-with-husky-2.0.0.tgz",
    "clean-working-directory": "git reset --hard HEAD && git clean -fdx",
    "remove-node-modules": "npx --yes rimraf@5.0.7 --glob ./**/node_modules",
    "clear-all-caches": "npx --yes clear-npx-cache && npm cache clean --force && npm cache verify",
    "pe": "npm run clean-working-directory && npm run remove-node-modules && npm run clear-all-caches  && echo '\n....Environment reset & Ready'"
  },
```

`pe` stands for prepare environment, depending on the complexity of your package, you may find yourself clearing the environment many times, like I did, trying to get the accurate directory to move the `.commitlintrc.json` file from my package folder in `node_modules` to the user's project repo. Preparing the environment allows you to treat the testing repo as if it were a user's project repo installing and using your package. Constantly resetting your environment to test your package and typing out `npm run pe` is less annoying when you are fixing a bug than `npm run prepare-environment` . Remember this is your test environment and not something intended for public consumption so be comfortable while you work and use whatever shorthand syntax is convenient for you.

`git clean -fdx`: Removes all untracked files and directories, including those ignored by `.gitignore`.

## That's All Folks

---

I hope you found this helpful, if you're curious about the video that piqued my interest and the package that resulted here they are: [The video](https://www.youtube.com/watch?v=2J9VnYiZ_Ts&ab_channel=remarkablemark), [My attempt](https://www.npmjs.com/package/commitlint-with-husky).