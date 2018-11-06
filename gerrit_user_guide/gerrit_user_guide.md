[clone_from_github]: ./images/clone_from_github.png
[clone_from_gerrit_a]: ./images/clone_from_gerrit_a.png
[clone_from_gerrit_b]: ./images/clone_from_gerrit_b.png
[download_a_patch_from_gerrit]: ./images/download_a_patch_from_gerrit.png
[gerrit_topic]: ./images/gerrit_topic.png


Table of Contents

- [Gerrit User Guide](#gerrit-user-guide)
  - [Clone a project](#clone-a-project)
    - [Cloning a project from github](#cloning-a-project-from-github)
    - [Cloning a project from gerrit](#cloning-a-project-from-gerrit)
  - [Upload a change](#upload-a-change)
    - [Pre requisites](#pre-requisites)
    - [Uploading the change](#uploading-the-change)
    - [Good practices](#good-practices)

# Gerrit User Guide

This is a Gerrit guide that is dedicated to Gerrit end-users. It explains the
standard Gerrit workflows and how a user can adapt Gerrit to personal
preferences.<br>

It is expected that readers know about Git and that they are familiar with
basic git commands and workflows.


## Clone a project

The project can be cloned directly from `github` or `gerrit`

### Cloning a project from github

1. Go to [github.intel.com](https://github.intel.com)
2. Select the project
3. Click on the button `Clone or download` and copy the command to clone the
project

![all_text][clone_from_github]

### Cloning a project from gerrit

- Go to `Gerrit`
- Select `Projects` > `List` and then select the desire project

![all_text][clone_from_gerrit_a]

- Copy the command to clone the project

![all_text][clone_from_gerrit_b]

## Upload a change

### Pre requisites

1 - gitreview

Before to upload a patch to Gerrit, the cloned project must have the `.gitreview`
configuration file in root, and the file must looks as:

```text
[gerrit]
host=<GERRIT_IP>
port=<GERRIT_PORT>
project=<CURRENT_PROJECT>
defaultbranch=<BRANCH>
```

2 - Gerrit as remote origin

Gerrit as remote origin must be configured in order to push/fetch changes

To configure it please type the following command:
```bash
$ git review -s
```

To check it please type the following command:
```bash
$ git remote -v
```

The output will be similar to this:
```text
gerrit	ssh://<USER>@<GERRIT_IP>:<GERRIT_PORT>/<GERRIT_PROJECT> (fetch)
gerrit	ssh://<USER>@<GERRIT_IP>:<GERRIT_PORT>/<GERRIT_PROJECT> (push)
origin	https://github.intel.com/<GITHUB_PROJECT>.git (fetch)
origin	https://github.intel.com/<GITHUB_PROJECT>.git (push)
```

### Uploading the change

To upload a change to Gerrit is required that the commit has a `Change-Id` on it
otherwise gerrit will refuse the patch<br>
Follow the next steps to upload a patch

1. `git status` :: to see/identify the untracked changes
2. `git add <file>` :: to add a file for commit it
3. `git commit` :: this command will open a editor in which you can write the
commit content, usually the content conformed by a title and a body and these
should not exceed of the 79 characters per line
4. `git log -1` :: this step is only to verify if the commit has the `Change-Id`
on it, otherwise please type `git commit --amend` to add it. The last command
will open again the commit message but this time will be enough with closed only
to get the `Change-Id`
5. `git review` :: this command will upload the change to Gerrit for revision

### Good practices

A good practice for upload changes is to make a new branch and start to work on
it, e.g:

- When a project is cloned usually the default branch is `master`, assuming that
we can do the following:

1. Make a new branch in order to keep `master` branch as clean as you can<br>
`git checkout -b <MY_NEW_BRANCH>` :: this command will create a new branch and
switch to it automatically 
2. Start to work on this branch
3. Once you finish your job on this branch, switch to master `git checkout master`
and create another branch if needed 

## Download a patch from Gerrit


### Pre requisites

### Downloading the patch
The reasons for download a patch from Gerrit could be the followings:

- Upload a new revision of the patch
- Test the patch locally 

1. Go to `Gerrit` and select to patch to download
2. Click on `Download` button<br>
The most used method to download the patch is by `Checkout` but you can use
whatever you want

![all_text][download_a_patch_from_gerrit]

3. Click on the desire download option to copy it to the clipboard
4. Over the repository cloned in you machine paste the copied in the previous
step
5. In this step is recommended to make a branch `git checkout -b <MY_BRANCH>`.
The same branch of the patch in Gerrit. You can find the patch's branch in Gerrit
under `Topic` tag

![all_text][gerrit_topic]