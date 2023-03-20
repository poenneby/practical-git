# Practical Git

## Prerequisites

### All you need is Git

This hands-on assumes you have a __Git__ client installed.

MacOS and Linux users will already have Git installed.

For Windows users we recommend installing __Git Bash__ which emulates a Linux shell with bash and Git installed.

## The beginning

Let's assume you are starting work on a new application. There will be code and you want to keep a version history of that code.

The application will output random numbers into a text file.

Let's start by creating a new directory for our application and initialise a git repository inside.


```
mkdir random-number-generator
cd random-number-generator
git init
```

From now on all the files that we add modify or delete in this directory will be traced by version control.

Next up we create the shell script. At first we will let it just output something to the console.

Open up your favourite text editor and paste the following code into file called `generate.sh`

```
#!/bin/bash

echo "Hello"
```

Save the file and make it executable

```
chmod 744 generate.sh
```

We can run it to ensure it output "Hello" as expected

```
> ./generate.sh
Hello
```

Check the status of our repository
```
git status
```

We can see our new file that is not yet added to the tracked files.

We have made some progress and it's time to commit it to the repository.

First add the file to the index
```
git add .
```
Then commit it with a descriptive message of our progress
```
git commit -m "Generating simple greeting"
```

Our first commit is visible by running `git log`

```
commit 6e98149c015338fae004300250a66cbea5e508a0
Author: Peter Onneby <ponneby@xebia.fr>
Date:   Tue Jun 26 21:48:37 2018 +0200

    Generating simple greeting
```

We get information about the author, a timestamp and the message. But perhaps most importantly our commit has a unique identifier, a SHA1

Let's move on to generating random numbers into a text file - change the `generate.sh` script to echo `$RANDOM` and write it to a file.

```
#!/bin/bash

echo $RANDOM > generate.out
```

Hey this application is really starting look great! Let's commit our progress. But before we do check the status

```
git status
```

We can see that `generate.sh` has been modfied but we also see a new untracked file, `generate.out`.
Something tells us version controlling generated files is a bad idea. Let's ignore this file.

Open up your favourite text editor again and create a new file called `.gitignore`

Inside it we add `*.out` which matches all files with the extension `out`.

Running `git status` again omits the generated file.

We add all the files to be committed and commit
```
git add .
git commit -m "Generating random numbers to a file"
```

Let's run `git log` again:

```
commit b1cae93d321b59d17c8f402f1d699f67e40631bd
Author: Peter Onneby <ponneby@xebia.fr>
Date:   Tue Jun 26 22:18:38 2018 +0200

    Generating random numbers to file

commit 6e98149c015338fae004300250a66cbea5e508a0
Author: Peter Onneby <ponneby@xebia.fr>
Date:   Tue Jun 26 21:48:37 2018 +0200

    Generating simple greeting
```

That's a lot of information but we are not necessarily interested in all of it, especially when we start accumulating a lot of commits.

`git log` takes a number of arguments that helps format the output. Try this:

```
git log --graph --decorate --pretty=oneline --abbrev-commit
```

```
* b1cae93 (HEAD -> main) Generating random numbers to file
* 6e98149 Generating simple greeting
```

Here we get a more brief summary of our commits and you will note that we only need the first 7 characters of the SHA1 to identify our commit.

However useful this command is, it's not very practical to type all that everytime we want to check our logs. Let's add an `alias` to get around this.
```
git config --global alias.lol "log --graph --decorate --pretty=oneline --abbrev-commit"
```

And now running `git lol` will give you the abbreviated log output! 🎉

Let's add another alias allowing us show all commits (branches etc)
```
git config --global alias.lola "log --graph --decorate --pretty=oneline --abbrev-commit --all"
```

## Share the work

Someone joins the team and you will need to share the code somehow.

### Simulate a remote repository locally

Run the following from the parent directory
```
mkdir random-number-generator.git && cd random-number-generator.git && git init --bare
```

Once you have got the repository you can add it as a remote like this

```
git remote add origin ../random-number-generator.git
```

And with the remote added we can push our local repository
```
git push -u origin main
```

The `-u` option configures our repository to set origin main as our default upstream thus pushing our future changes requires only a `git push`

Now our new team member can clone this repository and get to work on the next feature - __generating files into a directory__

```
git clone random-number-generator.git member
```

## Conflicts

To satisfy this new requirement we modify our script slightly:

```
#!/bin/bash

mkdir -p out
echo $RANDOM > out/generate.out
```

And to save a few keystrokes we can add and commit in one go
```
git commit -am "Generating files to out directory"
```

Our product owner has validated this feature so we push!

But in the meantime our collegue has decided to work on the next feature, __generating a named output file into a directory__

```
#!/bin/bash

mkdir -p out
echo $RANDOM > out/$1.out
```

He's pretty happy with that code and decides to commit and push it.

```
git commit -am "Generating named files to out directory"
git push
```

But his `push` is rejected, Git has protected us from overwriting work:

```
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'git@github.com:poenneby/random-number-generator.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
```

Let's `git pull` as suggested in the hints.
```
Auto-merging generator.sh
CONFLICT (content): Merge conflict in generator.sh
Automatic merge failed; fix conflicts and then commit the result
```
A `pull` is fetching and attempting to merge our code with the upstream in one go.

Ouch we now have a conflict to resolve! 🤬


We can solve the conflict by manually removing the code that generated to a static file.

Or alternatively we can `git checkout --ours generator.sh` to let git resolve the conflict for us by picking our version of the file.

Our file should look like this after:

```
#!/bin/bash

mkdir -p out
echo $RANDOM > out/$1.out
```

This conflict means we now have to add another commit, a merge commit.

```
git add .
git commit -m "Resolving conflict between generators"
```

Running `git lol` we observe that our log is somewhat messy having that branch ending with a useless merge commit

```
*   b605c8f (HEAD -> main) Resolving conflict between generators
|\
| * 3c0d104 (origin/main, origin/HEAD) Generating files to out directory
* | 1206773 Generating named files to out directory
|/
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```

### We can do better!

## Using a branch per feature

### Rewind

Let's first rewind our work back to the commit when there was only one person on the team
```
* b1cae93 Generating random numbers to file
```

_Please avoid doing this on your production code repositories!_

With the SHA1 of that commit we can reset the code as if this commit was the latest, and we also push force this modification of our commit log
```
git reset --hard b1cae93
git push --force
```

You will also have to reset your collegues code to be in sync with the modified origin

```
git fetch
git reset --hard origin/main
```

### Features

Developer 1 will work on the feature that generates files into the out folder (feature-1)
```
git checkout -b feature-1
```

Developer 2 will work on the feature that generates named files into the out folder (feature-2)
```
git checkout -b feature-2
```

Developer 1 is done and he commits his work in his branch and push the new branch upstream

```
git commit -am "Generating files to out directory"
git push -u origin feature-1
```

Developer 2 is done and he commits his work in his branch and push the new branch upstream
```
git commit -am "Generating named files to out directory"
git push -u origin feature-2
```

Developer 1 wants to merge his code to main but before he does he asks Developer 2 to review his work

So Developer 2 checkouts the `feature-1` branch and performs a diff to see what's changed compared to main:

```
git checkout feature-1
git diff main..feature-1
```

Developer 2 decides to accept `feature-1` to be merged to `main`

Developer 1 merge `feature-1` to `main`

```
git checkout main
git merge feature-1
git push
```

Developer 2 has also finished his development and asks Developer 1 to review the code.

Developer 1 decides to accept `feature-2` to be merged to `main`

But before merging he must ensure his branch is up to date with `main` to avoid merge conflicts

```
git fetch
git rebase origin/main
First, rewinding head to replay your work on top of it...
Applying: Generating named files to out directory
Using index info to reconstruct a base tree...
M	generator.sh
Falling back to patching base and 3-way merge...
Auto-merging generator.sh
CONFLICT (content): Merge conflict in generator.sh
error: Failed to merge in the changes.
Patch failed at 0001 Generating named files to out directory
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```
Rebasing the branch means including the commits from the source branch first and then applying the commits from the branch on top.

And it turns out there are conflicts to be resolved. The clear advantage is that we are resolving the conflict safely in our branch and not at `main`.

This time we resolve the conflict by accepting "their" modifications, "their" being "us" in our branch.

```
git checkout --theirs generator.sh
```

After verifying the file contents we add the resolved file and continue rebase
```
git add generator.sh
git rebase --continue
```

The branch `feature-2` is now up to date with main and we can simply merge the branch to main now.

```
git checkout main
git merge feature-2
git push
```

We finish up by deleting the merged branches both locally and remotely

```
git branch -D feature-1
git push origin --delete feature-1
git branch -D feature-2
git push origin --delete feature-2
```

Running another `git lol` from `main` we observe a clean single tree history

```
* da61890 (HEAD -> main, origin/main) Generating named files to out directory
* 18fa162 Generating files to out directory
* b1cae93 Generating random numbers to file
* 6e98149 Generating simple greeting
```



