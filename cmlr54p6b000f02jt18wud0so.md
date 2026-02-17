---
title: "How I Use Git Worktrees Without Breaking Everything"
seoTitle: "Mastering Git Worktrees Safely"
seoDescription: "Use Git worktrees for efficient branch management, avoiding stashing and context switching, and streamline your workflow"
datePublished: Tue Feb 17 2026 21:54:03 GMT+0000 (Coordinated Universal Time)
cuid: cmlr54p6b000f02jt18wud0so
slug: how-i-use-git-worktrees-without-breaking-everything
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1771364967755/bfd8e9f5-aa06-4687-b5c4-d70d9a8b728e.png
tags: terminal, workflow, git, branching, worktrees

---




## Why Context Switching in Git Is More Painful Than It Needs To Be

You're deep into a feature branch. Tests are half-written, there are uncommitted changes everywhere, and your teammate pings you - urgent hotfix needed on `main`. Now what?

You stash your work, hope you remember to pop it, switch branches, fix the bug, switch back, pop the stash, and pick up where you left off. If you're lucky, nothing explodes.

There's a better way. **Git worktrees let you check out multiple branches into separate folders at the same time** - all sharing the same repository. No stashing. No context loss. No second clone eating up your disk.

This post covers how to set them up correctly, what to avoid, and the one trap that catches almost everyone who tries the bare repo approach.

---

## Step 1: Understand What a Worktree Actually Is

A worktree is an additional working directory linked to your existing repository. Every worktree shares the same `.git` folder - the same history, the same remotes, the same commits - but each has its own branch checked out independently.

Think of it like having multiple desks in one office. Each desk has a different version of the project open. But there's only one filing cabinet that all desks share.

Rather than scattering worktree folders everywhere, a cleaner approach is a dedicated `worktrees/` folder sitting alongside your repos:

```plaintext
~/Repositories/
  myproject/                        ← main repo
  another-project/                  ← another main repo
  worktrees/
    myproject/
      feature-login/                ← worktree
      hotfix-payment/               ← worktree
    another-project/
      feature-dark-mode/            ← worktree
```

This keeps repos and worktrees visually separated, scales cleanly across multiple projects, and means your worktrees are never accidentally committed or wiped.

**One-time setup** - create the `worktrees/` folder once:

```bash
mkdir ~/Repositories/worktrees
```

Two rules to remember before you touch anything:

* Worktrees should live **outside** the main repo folder

* Use a **normal clone**, not `git clone --bare` (more on this below)

---

## Step 2: Add Your First Worktree

Typing the full path every time gets old fast. Add this alias to your `.gitconfig` once:

```bash
# add worktree to ../worktrees/<repo-name>/<branch-name>
git config --global alias.wt-add '!f() { mkdir -p ../worktrees/$(basename $(pwd)) && git worktree add ../worktrees/$(basename $(pwd))/$1 $1; }; f'
```

Now, from inside any repo, adding a worktree is one command:

```bash
cd ~/Repositories/myproject
git wt-add feature-login
```

That automatically creates `../worktrees/myproject/feature-login/` with `feature-login` checked out. The `mkdir -p` handles creating the project subfolder if it doesn't exist yet - so the first worktree for any project just works.

**One important rule:** the branch you pass to `wt-add` cannot be currently checked out. Git won't let the same branch exist in two places at once. This means if you want to create a new branch and immediately add it as a worktree, you must switch off it first:

```bash
git checkout -b feature-login   # create and switch to new branch
git checkout main               # switch back so the branch is free
git wt-add feature-login        # now add it as a worktree
```

If you skip the `git checkout main` step, `wt-add` will fail because `feature-login` is already checked out in your current session.

To add a worktree **and create a new branch** in one step without the back-and-forth, use the native command directly:

```bash
#git worktree add -b <new-branch-name> <worktree-location>
git worktree add -b hotfix-payment ../worktrees/myproject/hotfix-payment
```

---

## Step 3: List and Manage Worktrees

Check what worktrees you have active at any time:

```bash
git worktree list
```

Output:

```plaintext
/home/you/Repositories/myproject                           abc1234 [main]
/home/you/Repositories/worktrees/myproject/feature-login   def5678 [feature-login]
/home/you/Repositories/worktrees/myproject/hotfix-payment  ghi9012 [hotfix-payment]
```

Remove a worktree when you're done with it:

```bash
git worktree remove ../worktrees/myproject/hotfix-payment
```

If you deleted the folder manually instead of using the command, clean up the stale reference with:

```bash
git worktree prune
```

---

## Step 4: Use It Like Normal

This is the part people don't believe until they try it. From inside any worktree folder, all your standard git commands work exactly as expected:

```bash
cd ../worktrees/myproject/feature-login

git fetch origin
git rebase origin/main
git push origin feature-login
```

Remote tracking works. `origin/main` exists. `git pull`, `git rebase`, `git merge` - all normal. This is because worktrees inherit the full remote configuration from the main clone.

---

## The Bare Repo Trap

You'll find plenty of guides recommending `git clone --bare` as the "proper" way to use worktrees. The structure looks elegant:

```plaintext
project/
  project.git/      ← bare repo (no files, just git internals)
  main/             ← worktree
  feature/          ← worktree
```

**I would not recommend this for local development.** `git clone --bare` uses a different fetch refspec, which prevents remote-tracking branches from being created.

The workaround requires manually resetting the refspec, re-fetching, and deleting all local branch heads. It's fragile, poorly documented, and not worth the trouble for everyday development.

The accepted answer on [Stack Overflow](https://stackoverflow.com/a/54408181) explicitly recommends against `git clone --bare` for this use case.

## Why This Happens?

When you run a normal clone:

```bash
git clone <url>
```

Git sets this fetch refspec:

```plaintext
+refs/heads/*:refs/remotes/origin/*
```

That means:

* Remote branches are stored as **remote-tracking branches**
* They live under `refs/remotes/origin/*`
* You can reference them as `origin/main`, `origin/feature`, etc.

This is what makes commands like:

```bash
git rebase origin/main
```

work naturally.

But when you run:

```bash
git clone --bare <url>
```

Git changes the fetch refspec to:

```plaintext
+refs/heads/*:refs/heads/*
```

Now remote branches are written directly into:

```plaintext
refs/heads/*
```

There is no `refs/remotes/origin/` namespace at all.

In other words, the repository stops maintaining remote-tracking branches. It treats fetched branches as local branches instead.

---

## The Practical Consequence

Because `refs/remotes/origin/*` is never created:

* `origin/main` does not exist
* `origin/feature` does not exist
* Commands that rely on remote-tracking refs fail

So when you run:

```bash
git rebase origin/main
```

Git is correct to say:

```
fatal: invalid upstream 'origin/main'
```

That reference was never created.

## Short Term Fix?

On a repo by repo basis you would now have to add the command:

```bash
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*" 
```

This restores remote-tracking branches, but you are now overriding the default behaviour of a bare clone.

## Can you make this Global?

Well if you are not a Git Guru, you might surmise you can fix this globally.

A few things I'd like to clear up:

A bare repository is designed to be a **central repository**, not a working repository.

Central repositories:

- Do not check out branches

- Do not maintain remote-tracking branches

- Usually receive pushes

- Do not need `refs/remotes/origin/*` intentionally

Technically, you can run:

```bash
git config --global remote.origin.fetch ...
```

 When cloning, Git writes a `[remote "origin"]` section into the repository’s local config, including its own fetch refspec:

```text
[remote "origin"]
    url = ...
    fetch = +refs/heads/*:refs/remotes/origin/*
```

Because local configuration overrides global configuration, a globally defined `remote.origin.fetch` will not affect repositories created with `git clone`.

```text
system < global < local
```

If you insist and really want to use `--bare` repos with future repo accommodation you will need some automation, [Morgan Cugerone](https://morgan.cugerone.com/) has one of the more [elegant solutions](https://archive.is/UmAsb#50%) I have found to tackle this development style. As you can see I opted for a simpler workflow.

---

## Common Worktree Issues

**"cannot checkout - already checked out"** You tried to add a worktree on a branch that's already checked out somewhere else. Git prevents this. Create a new branch instead:

```bash
git worktree add -b new-branch ../worktrees/myproject/new-branch
```

**The worktree folder shows up in git status.** You put the worktree inside the repo folder. Move it outside, or add the path to `.git/info/exclude` (not `.gitignore` - you don't want this tracked).

**git worktree list shows a prunable worktree.** You deleted the folder manually. Run `git worktree prune` to clean up the reference.

**node\_modules missing in new worktree.** Each worktree needs its own dependency install. Dependencies are not shared. Run `npm install` (or equivalent) in each worktree folder.

---

## Summary

| Task                             | Command                                                       |
| -------------------------------- | ------------------------------------------------------------- |
| Add worktree for existing branch | `git wt-add branch-name` (my solution)                        |
| Add a worktree with a new branch | `git worktree add -b new-branch ../worktrees/repo/new-branch` |
| List all worktrees               | `git worktree list`                                           |
| Remove a worktree                | `git worktree remove ../worktrees/repo/branch`                |
| Clean up deleted worktrees       | `git worktree prune`                                          |

---

## Final Thoughts

Worktrees are one of those features that sound complicated until you use them once, after which you wonder how you worked without them. Create a `worktrees/` folder, add the `wt-add` alias, and from that point on, every new branch is one command away from having its own isolated workspace.

The one thing to remember: normal clone, folders outside the repo, and stay away from `git clone --bare` unless you enjoy fixing refspecs.

**The rule:** If you're reaching for `git stash` because of an interruption, reach for `git worktrees` instead. I've created some nifty `git stash` aliases but I doubt I'll be using them anymore.
