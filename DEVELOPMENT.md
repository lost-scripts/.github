---
ScriptBirth = "20240428-2142"
ScriptBuild = "20240503-0447"
---

<h1 align="center">Lost Script Creation & Workflow</h1><br>

### 1\. Creating a new script

* 1.1\. Starting from a kind of _super-repository_ _[ls][1]_ containing all project's common and shared files in e.g.:
 	<br>
	<details>
		<summary>D:/Rai/Projects/Moho/LS/ (Click to expand)</summary>

		📂ls
		│   .gitattributes
		│   .gitignore
		│   README.md
		│
		├───📂.git
		│
		├───📂docs
		│   │   index.htm
		│   └───📂assets
		│           README_icon.png
		│           README_logo.png
		│           README_overview_001.png
		│
		├───📂Menu
		│   │   ls_separator.lua
		│   │
		│   └───📂- Lost Scripts
		│           ls_webpage.lua
		│
		├───📂Modules
		│       ls_gui.lua
		│       ls_modules.lua
		│
		├───📂ScriptResources
		│   └───📂ls
		│           logo.png
		│
		├───📂Tool
		│       _tool_list_ls.txt
		│
		└───📂Utility
				ls_utilities.lua
	</details>
	<br>

	And assuming it has a `main` and a `dev` branch [^1] as well as having set up the appropriate environment [^2], proceed to make a clone of it (these relative paths assume you are executing the commands from the repo to clone) as follows:

	```bash
	git clone --depth 1 -b ls_my_script file://D:/Rai/Projects/Moho/LS/ls ../ls_my_script # --depth requires such kind of absolute path
	```

	> :memo: **Note:** The purpose of not just using e.g. `git clone -b main ../ls ../ls_my_script` [^3] is limit history to the very last commit.  

	As for now, the remotes for the _ls_my_script_ repo should look like this:

	```bash
	git remote -v
	origin  file://D:/Rai/Projects/Moho/LS/ls (fetch)
	origin  file://D:/Rai/Projects/Moho/LS/ls (push)
	```

	Rename it and, for convenience, **make its path relative**:

	```bash
	git remote rename origin ls
	```
	```bash
	git remote set-url ls ../ls # Now the remote ls relatively points to the local super-repo "ls" 
	```

	So it end up looking like this:

	```bash
	git remote -v
	ls  ../ls (fetch)
	ls  ../ls (push)
	```

* 1.2\. From the new script repo, add the script's own files like _ls_my_script.lua_ and so, and **remove any shared files it may not need**, either by means of the explorer or the console, e.g.:

	```bash
	cd ../ls_my_script
	git rm -r Modules # Should not using any of its contents, directly delete "Modules" folder
	git rm Utility/ls_utilities_ext.lua # Delete only the unused ls_utilities_ext.lua
	ln -s ScriptResources/ls_my_script/docs docs # If necessary (or: mklink /d docs ScriptResources\ls_my_script\docs for creating the symlink from CMD)
	git add -A # Similarly to "addremove", adds (even untracked) & removes files (if necessary, use: git add -u instead to add only deleted files)
	git commit -m "ls_my_script: Initial commit"
	```

* 1.3\. Now your script repository is ready for start working locally. To host it on GitHub, create a namesake repository there and follow the suggested instructions:

	```bash
	git remote add origin https://github.com/lost-scripts/ls_my_script.git
	git branch -M main # Not necessary if you already use main...
	git push -u origin main
	```

<br>

---

### 2\. The workflow

* 2.1\. Two ways for bringing changes in the _super-repo_ "ls" to the script repo:

	```bash
	git pull ls main # Bring changes and merge all at once (recommended)
	```
	```bash
	git fetch ls main # Bring changes only, so then you can do "git diff ...ls/main" (or git diff ..ls/main file-name) to see changes before doing "git merge ls main"
	```

	> :warning: **Warning:** In any case, all possible conflicts will have to be resolved. To delete all unnecesary common files detected as _deleted by us_ at once, you can use the following command:

	```bash
	git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {} # Or its alias: grdu
	```

* 2.2\. Create a new repository for the script on GitHub (or the service of your choice). Then, as usual, add it as remote named _origin_ and upload:

	```bash
	git remote add origin https://github.com/lost-scripts/ls_my_script.git
	git push -u origin main
	```

	The script's remotes should look like this:

	```bash
	git remote -v # Example of how the remotes would look like:
	ls    ../ls (fetch)
	ls    ../ls (push)
	origin  https://github.com/lost-scripts/ls_my_script.git (fetch)
	origin  https://github.com/lost-scripts/ls_my_script.git (push)
	```

* 2.3\. Before uploading local changes to the remote, it's always advisable to make sure that the script repo is up to date:

	```bash
	git pull origin main # To bring any possible changes from the script repo on GitHub
	```

	```bash
	git pull ls main # To bring any possible changes in common/shared files from an up-to-date local super-repo "ls"
	```

* 2.4\. Now, local changes can be pushed safely to the remote repository of the script on GitHub as usually:

	```bash
	git push origin main
	```

	> :memo: **Note:** When the time comes to create a release, finish off the `.github/README.md` file and export it so that the resulting HTML file end up like this: `ScriptResources/ls_my_script/README.html`.

<br>

---

### 3\. Making a release

* 3.1\. Finally, when everything is ready for launching a release, start for creating an _annotated tag_:

	```bash
	git tag -a v1.0.0 -m "Message for version 1.0.0" # Possible messages: Initial release, Bug fixes and feature X, etc.)
	```
	```bash
	git push origin v1.0.0 # Upload tags to remote (or `git push origin --tags` for uploading all unuploaded tags)
	```

	Then, go to the repository page on GitHub, click on _Releases_ or _Tags_ section and, next to the tag you want to convert into a release, click on _Create release_. Once there, fill out the release information:

	- **Tag version**: Select the just uploaded tag.
	- **Release title**: Set a title for the release.
	- **Describe this release**: Adds notes on changes, improvements and fixes included in this release.
	- **Attach Binaries** (if necessary): Drag & drop binary or executable files.
	- **Publish Release**: Once all the information is filled in, click on _Publish release_.

* 3.2\. Alternatively, it's always possible to create a _.zip_ archive of the script in question in one of the following ways:

	```bash
	git archive -o ls_my_script.zip main # Simplest (equate to: git archive --format zip --output ls_my_script.zip main)
	```
	```bash
	git archive --o "$(basename "$(git rev-parse --show-toplevel)").zip" main # Or the more advanced (with smart folder creation, autoname & alias prone e.g. garc): p mkdir -p _releases && git archive -o _releases/$(basename "$(git rev-parse --show-toplevel)").zip main
	```

	> :memo: **Note:** To exclude elements, create a .gitattributes file in the root of the script repo and put on it e.g.:
	>```
	>.gitattributes export-ignore
	>.gitignore export-ignore
	>/.github export-ignore
	>```
	><br>

<br>

---

### 4\. Pages Deployment

* 4.1\. Any time GitHub Pages needs to be updated and assuming a dummy `gh-pages` branch exists, do:

	```bash
	git checkout gh-pages
	git reset --hard main # Ensure gh-pages is equal to main, where the real changes should be, to ease things along
	git push origin gh-pages -f # Force the just-reset-branch push to remote
	git checkout main # Return to main branch (or whichever you were) to avoid start working on gh-pages by mistake!
	```

	Alternatively, one can just use alias `deploy` and confirm. In any case, all this ensures deployments are only made when necessary and without having to worry about possible merge conflicts and such...

	> :warning: **Warning:** That's all the `gh-pages` branch is for and no work should ever be done on it or it will be irremediably lost.

<br>

[^1]: Current development should be done in _dev_ branch or derivates and merged with _main_ once significant progress has been made and tested. But more on this later...

[^2]: Based on settings suggested by the [ENVIRONMENT.md](./blob/main/DEVELOPMENT.md) file. But more on this later too...

[^3]: `-b main` specifies the branch to clone (if omitted, the default one is cloned), `ls` is the name of repository's remote (or, if not applicable, its path) and, finally, `ls_my_script` is the name of the directory where the repository will be cloned.

[1]: <https://github.com/lost-scripts/ls> 'Go to "ls" super-repository on GitHub'