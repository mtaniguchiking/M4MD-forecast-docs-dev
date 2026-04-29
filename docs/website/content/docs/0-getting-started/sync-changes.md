---
title: Sync changes
weight: 4
bookToc: true
---

# Sync changes

## Checking for updates within RStudio
For many National Park Service users, getting the latest updates and pushing changes is easiest within RStudio. All members of NPS-SWNC/M4MD can get the latest updates. You must have write privileges to push your changes to the repo. Please send an email to [Cheryl McIntyre](mailto:cheryl_mcintyre@nps.gov) to request write access. To sync changes, begin by opening the M4MD RStudio Project File. 

To get the latest updates: 

  - Click on the Git tab
  - In the Git tab, click on the Pull button (blue down arrow)
  
To push your changes to the M4MD repo:

  - Click on the Git tab
  - In the Git tab, click on the Commit button (paper with green check mark)
  - In the Review changes popup window, stage one or more files for commit by clicking the check box next to the filename(s)
  - Enter a brief description of your changes in the Commit message box.
  - Click the Commit button
  - In the Git tab, click on the Push button (green up arrow)

### Troubleshooting
You try to pull the latest changes but fail with a warning that you have local uncommitted changes that would be overwritten. In this case, you can revert (lose all of your changes), commit your changes, or use git stash to stash your changes. Then you can pull again, which will fetch the remote branch and merge it with your local version.

You try to push your commits but the push fails. This often happens if there are commits on the remote branch that are not on your local version. Your first step is to pull the remote branch and then try pushing your local branch again.

You tried that and got a message about merge conflicts. This happens if you and someone else made different changes to the same line of code. You should be able to edit the problematic file, using the markers that indicate the conflict as your guide. Save your updated file, stage, and commit. Carefully review what you committed before you push. Sometimes weird things happen when you are resolving a merge conflict and you can push files that you did not intend to. Once you are statisfied, push your changes. 

## Checking for updates with a terminal

### Project updates
To get the latest __models-for-missing-data__ code, navigate to your project directory using a terminal, and run `git pull`.

### Submodule updates / individual submodules
```sh
cd <submodule path> # e.g., model-api
git fetch
git checkout gh-submodule
git pull origin gh-submodule
```

### Submodule updates / all submodules simultaneously
First, be sure to do any necessary housekeeping (remove, stash, or commit changes). To get the latest updates for each of the submodules, run:
```sh
git submodule update --recursive --remote
```
<!-- git submodule update --checkout -->
<!-- git submodule update --remote docs/website/themes/hugo-cite -->
If you encounter an error, it's likely you've made changes locally that are not yet saved (staged and committed using `git add` and `git commit`). Git won't replace changes in uncommitted files with changes on the remote by default. This is desirable behavior. Try committing your changes locally _before_ syncing with the remote.

### Pushing local changes to a submodule to its remote
First, `cd` into the submodule directory. We're going to do all of our Git work within the context of the submodule. If you've got uncommitted changes, do any necessary housekeeping:

{{< notice note >}}
If you're seeing files after running `git status` that you _don't_ want to commit to the version history, open the ".gitignore" file and add a new entry for the file you want to ignore. You can open ".gitignore" with any text editor, including RStudio. 
{{< /notice >}}

```sh
git status
git fetch
git checkout gh-submodule
git add .
git commit -m "<some descriptive message about your changes>"
git push origin gh-submodule
```
As always, please ensure you're not staging / committing unwanted files (e.g., binary files).

### Troubleshooting
If, on an attempt to pull from the remote submodule, you got `fatal: Not possible to fast-forward, aborting`, try:
```sh
git merge origin/<submodule>
```

<!-- ### Checking out a specific ref / branch as opposed to a specific commit
```sh
git fetch --all
git checkout gh-submodule
```

<!-- ### A special note to NPS users
You may need to disconnect from your VPN. -->
