## 5 Things I Learned Trying To Publish A Simple NPM Package



## Preface

While this is not my first published NPM package, I learned some cool tips that took me way too long to find/troubleshoot this time around so I thought I’d share them with you. I’d also like to preface this post by saying I am a Windows user, so some situations may be non-existent for Mac/Linux users.

## Introduction

As I mentioned above, this is not my first package. I did make a package called [fast-form-validator](https://www.npmjs.com/package/fast-form-validator) as a way to streamline form validation for simple websites. This time, however, I came across a YouTube video discussing linting your git commits using Husky for easier implementation of git hooks, in my case a pre-commit hook and Commitlint to ensure that my commits are in line with [conventional commits' Standards](https://www.conventionalcommits.org). The process for setting up the two modules was not long at all. I was simply curious If I could implement a single module that would make the set-up process faster and easier just for the challenge. I had seen this done before by other developers and I simply wanted to try it for myself.

### 1: Windows users install git for windows and take advantage of bash scripting!
---

Git for Windows provides a BASH emulation used to run Git from the command line. This is great since a lot of times we are given instructions intended for mac users that won't work on Windows machines. This was a real pain to pin down because I thought my scripts were running with git bash out of the box but kept getting error messages akin to Command Prompt instead. So, ensure to set the shell you want to be used when you call NPM:


```
npm config set script-shell "C:\\Program Files (x86)\\git\\bin\\bash.exe"
```



or (64bit installation)

```
npm config set script-shell "C:\\Program Files\\git\\bin\\bash.exe"
```

### 2: You don’t have to publish your package to NPM to test if your package works!
---

Of course, there had to be an easier way to test your package locally, with my first package I was so eager to see working code that I didn't even stop or slow down to discover the simpler way I know now.

NPM and NPX can both be executed on local repositories!

%[https://giphy.com/gifs/5VKbvrjxpVJCM]

```
npm install 'path/to/repo'
```

and to simulate an npm package, you can simply run:

```
npm pack
```

This will create the gzipped tar file (your-package-name-versionNumber.tgz) for you to reference.

So in the end it should look something like this:

```
npm install 'path/to/my-package/my-package-1.0.0.tgz
```

### 3:NPX cache on windows does not clear properly!
---

Whew! This one was a doozie to pin down. If you are implementing npx in your package, updates won't show on successive installs and you would need to clear the cache each time. Thankfully at some point, I noticed the error messages had references to code I was sure I changed and I figured it was not clearing with the usual `npm cache clean –force` and in some cases, the files that are located on windows at `%LocalAppData%/npm-cache/_npx` were locked and didn't allow me to manually delete them.
Ironically, there is a package that will delete it for you:

```
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

This time around I made sure to work smart and not hard and so I created a test repo (mine is called test-dummy) to install my package and ensure it behaved as I intended. Since I would constantly have to reset the test environment on each update, I quickly realized I can just use npm scripts in the dummy's package.json to make my workflow easier.

These scrips:
- reset the environment by undoing the changes (installation of my package) using git.
- remove the node_modules folder
- clear the npx cache folder
- print a message to the screen that the environment was reset
- installed my package via npm 
- installed my package via npx

This is how it looks:
```json
"scripts": {
    "test": "user scripts here",
    "ins": "npm install -D ../commitlint-with-husky/commitlint-with-husky-1.0.10.tgz",
    "ut": " git undo && npx rimraf ./**/node_modules && npx --yes clear-npx-cache && echo '\n....Environment Cleaned'",
    "ex": "npx --yes ../commitlint-with-husky/commitlint-with-husky-1.0.10.tgz -D"
  },
```
ins = install, ut= undo this, ex = execute npx

## That's All Folks
---

I hope you found this helpful, if you're curious about the video that piqued my interest and the package that resulted here they are: [The video](https://www.youtube.com/watch?v=2J9VnYiZ_Ts&ab_channel=remarkablemark), [My attempt](https://www.npmjs.com/package/commitlint-with-husky).
