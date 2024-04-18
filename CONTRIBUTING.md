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
		├───📂.github
		│   └───📂Docs
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

	And assuming it has a `main` and a `dev` branch [^1], proceed to make a clone of it (these relative paths assume you are executing the commands from the repo to clone) as follows:

	```bash
	git clone --depth 1 -b ls_my_script file://D:/Rai/Projects/Moho/LS/ls ../ls_my_script # --depth requires absolute paths in such format
	```

	:memo: **Note:** The goal of not simply use e.g. `git clone -b main ../ls ../ls_my_script` [^2] is limit history to the very last commit.  

	As for now, the remotes for the _ls_my_script_ repo should look like this:

	```bash
	git remote -v
	origin  file://D:/Rai/Projects/Moho/LS/ls (fetch)
	origin  file://D:/Rai/Projects/Moho/LS/ls (push)
	```

	Rename it and, for convenience, <u>make its path relative</u>:

	```bash
	git remote rename origin ls
	```
	```bash
	git remote set-url ls ../ls 
	```

	So it end up looking like this:

	```bash
	git remote -v
	ls  ../ls (fetch)
	ls  ../ls (push)
	```

* 1.2\. From the new script repo, add the script's own files like _ls_my_script.lua_ and so, and <u>remove any shared files it may not need</u>, either by means of the explorer or the console, e.g.:

	```bash
	cd ../ls_my_script
	git rm -r Modules # Should not using any of its contents, directly delete "Modules" folder
	git rm Utility/ls_utilities_ext.lua # Delete only the unused ls_utilities_ext.lua
	git add -A # This adds (even untracked) & removes files, is equivalent to "addremove" (if necessary, use: git add -u instead for adding only deleted files)
	git commit -m "ls_my_script: Initial commit"
	```
---

### 2\. Workflow

* 2.1\. Two ways for bringing changes in the _monorepo_ "ls" to the script repo:

	```bash
	git pull ls main # Bring changes and merge all at once (recommended)
	```
	```bash
	git fetch ls main # Bring changes only, so then you can do e.g. "git diff ...ls/main" (or git diff ..ls/main file-name) to see changes before doing "git merge ls main"
	```

	:memo: **Note:** In any case, all possible conflicts will have to be resolved. To delete all files detected as _deleted by us_ at once, you can use the following command:

	```bash
	git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {} # Or its alias: grmu
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
	git pull ls main # To bring any possible changes in common/shared files from the, equally up-to-date, local monorepo "ls"
	```

* 2.4\. Now, local changes can be pushed safely to the remote repository of the script on GitHub as usually:

	```bash
	git push origin main
	```

	:memo: **Note:** When the time comes to create a release, finish off the `.github/README.md` file and export it so that the resulting HTML file end up like this: `ScriptResources/ls_my_script/README.html`.
---

### 3\. Making a release

* 3.1\. Finally, when everything is ready for launching a release, start for creating an _annotated tag_:

	```bash
	git tag -a v1.0.0 -m "Message for version 1.0.0" # Possible messages: Initial release, Bug fixes and feature X, etc.)
	```
	```bash
	git push origin v1.0.0 # Upload tags to remote (or `git push origin --tags` for uploading all unuploaded tags)
	```

	Then go to the repository page on GitHub, click on _Releases_ or _Tags_ section and, next to the tag you want to convert into a release, click on _Create release"_. Once there, fill out release information:

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
	git archive --o "$(basename "$(git rev-parse --show-toplevel)").zip" main # Advanced (ensure destination folder exists, autoname & allow alias e.g. garc): p mkdir -p _releases && git archive -o _releases/$(basename "$(git rev-parse --show-toplevel)").zip main
	```

	:memo: **Note:** To exclude some elements, create a .gitattributes file in the root of the script repo and put on it e.g.:

	```
	.gitattributes export-ignore
	.gitignore export-ignore
	/.github export-ignore
	```

[^1]: Current development should be done in _dev_ branch and merged with _main_ once significant progress has been made and testes. But more on this later...

[^2]: `-b main` specifies the branch to clone (if omitted, the default one is cloned), `ls` is the name of repository's remote (or, if not applicable, its path) and, finally, `ls_my_script` is the name of the directory where the repository will be cloned.

[1]: <https://github.com/lost-scripts/ls> 'Go to "ls" super-repository on GitHub'
