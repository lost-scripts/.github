<h1 align="center">Development</h1><br>

### 1\. Creating a new Lost Script

* 1.1\. Starting from a kind of _super-repository_ _[ls][1]_ containing all project's common and shared files in e.g.:
 	<br>
	<dl><dd>
	<details>
		<summary><b>D:/Rai/Projects/Moho/LS/</b> (Click to expand)</summary>
 		<br>

		📂ls
		│   📄.gitattributes
		│   📄.gitignore
		│   🔗README.md (ScriptResources/ls/docs/index.htm)
		│
		├───📂.git
		│
		├───🔗docs (ScriptResources/ls/docs)
		│
		├───📂Menu
		│   │   📄ls_separator.lua
		│   │
		│   └───📂- Lost Scripts
		│           📄ls_webpage.lua
		│
		├───📂Modules
		│       📄ls_modules.lua
		│
		├───📂ScriptResources
		│   └───📂ls
		│       └───📂docs
		│               📄index.css
		│               📄index.htm
		│               📄index.html
		│               📄index.js
		│               🖼index_favicon.svg
		│               🖼index_...
		│
		│           🖼logo.png
		│
		├───📂Tool
		│       📄_tool_list_ls.txt
		│
		└───📂Utility
				📄ls_utilities.lua
	</details>
	</dd></dl>

	And assuming it has a `main` and a `dev` branch [^1] plus, preferably, the appropriate environment [^2] has been set up, proceed to make a clone of it (these relative paths assume you are executing the commands from the repo to clone [^3]) as follows:

	```bash
	git clone . ../ls_my_script -o ls # Alternatives: `git clone ../ls ../ls_my_script -o ls` or `git clone -b <branch> . ../ls_my_script -o ls` (for cloning another branch than default/main). The `-o ls` is for directly renaming the remote `origin` to `ls`
	```

	> :memo: **Note:** Initially, recommendation was use `git clone --depth=1 -b main file://D:/Rai/Projects/Moho/LS/ls ../ls_my_script` (--depth requires such kind of absolute path) in order to limit history to the very last commit (resulting in a _shallow repo_), but turns out then Git rejects to push to a new repo, so ruled out as of today...

	As for now, the remotes for the _ls_my_script_ repo should look something like this:

	```bash
	cd ../ls_my_script
	git remote -v
	ls  file://D:/Rai/Projects/Moho/LS/ls/. (fetch)
	ls  file://D:/Rai/Projects/Moho/LS/ls/. (push)
	```

	Optionally, should for convenience you preferred a relative path, just use: `git remote set-url ls ../ls`, and the remote will end up as follows:

	```bash
	git remote -v
	ls  ../ls (fetch)
	ls  ../ls (push)
	```

* 1.2\. From the **root** of the new repo (otherwise suggested paths should be adjusted in consequence):
 
	Add the script's own files like _ls_my_script.lua_, _ScriptResources\ls_my_script\docs.index.htm_ and so, and (recomendably) **remove any shared files the new script may not need**, either via the explorer or the console, e.g.:

 	```bash
	git rm -r Modules # Should not using any of its contents, directly delete "Modules" folder
	git rm Utility/ls_utilities_ext.lua # Delete e.g. only the unused ls_utilities_ext.lua
	```

	Update symlinks by simply running the function `updateall ls_my_script` or manually (although Bash may require to run `export MSYS=winsymlinks:nativestrict` before) by:

	```bash
	rm -f docs; ln -s ScriptResources/ls_my_script/docs docs # Recreate "docs" symlink, also by the function `updatedocs ls_my_script` (or delete it and from CMD: mklink /d docs ScriptResources\ls_my_script\docs)
	rm -f README.md; ln -s ScriptResources/ls_my_script/docs/index.htm README.md # Recreate "README.md" symlink, also by the function `updatereadme ls_my_script` (or delete it and from CMD: mklink README.md ScriptResources\ls_my_script\docs\index.htm)
	rm -rf ScriptResources/ls/docs; ln -s ../../../ls/ScriptResources/ls/docs ScriptResources/ls/docs # Create "ScriptResources/ls/docs" symlink, also by the function `updatelsdocs` (or delete it and from CMD: mklink /d ScriptResources\ls\docs ..\..\..\ls\ScriptResources\ls\docs)
 	```

	> :bulb: **Tip:** To preserve symbolic links as-is in your backups with e.g. [_7-Zip_](https://www.7-zip.org/) or [_PeaZip_](https://peazip.github.io/), use _tar_ (or similar) as archive format and make sure that _Store symbolic links_ and _Store hard links_ options are checked. If using [_WinRAR®_](https://www.win-rar.com/) and _RAR5_ format instead, also make sure the advanced option _Allow absolute paths in symbolic links_ is checked at restoring. All that ensures symlinks will be restored as actual, and error-free, links.

	Make sure the _.gitattributes_ file counts with the lines listed bellow:
 	<br>
	<dl><dd>
	<details>
		<summary><b>.gitattributes</b> (Click to expand)</summary>
 		<br>

		.gitattributes merge=ours
		.gitignore merge=ours
		README.md merge=ours
		LICENSE.md merge=ours
		/docs merge=ours
		/ScriptResources/ls/docs merge=ours

		.gitattributes export-ignore
		.gitignore export-ignore
		LICENSE.md export-ignore
		README.md export-ignore
		/.github export-ignore
		/docs export-ignore
		/ScriptResources/ls/docs export-ignore

	> :memo: **Note:** In the first block, `merge=ours` strategy is applied to those files in common which modified contents must prevail over the ones in the super-repo. In the second, `export-ignore` is applied to elements that must be excluded from [releases](#3-making-a-release).
	</details>
	</dd></dl>


	And, lastly, add/remove everything and perform new repo's very first commit:

 	```bash
	git add -A # Similarly to "addremove", adds (even untracked) & removes files (if necessary, use: git add -u instead to add only deleted files)
	git commit -m "ls_my_script: Initial commit"
	```

* 1.3\. Now your script repository is ready for start working locally. To host it on GitHub (or the service of your choice), create a namesake repository there and follow the suggested instructions:

	```bash
	git remote add origin https://github.com/lost-scripts/ls_my_script.git
	git branch -M main # Not necessary if you already use main...
	git push -u origin main
	```

	After that, remotes for the script should look like this:

	```bash
	git remote -v # Example of how the remotes would look like:
	ls    ../ls (fetch)
	ls    ../ls (push)
	origin  https://github.com/lost-scripts/ls_my_script.git (fetch)
	origin  https://github.com/lost-scripts/ls_my_script.git (push)
	```

<br>

---

### 2\. Workflow

* 2.1\. The are basically two ways for bringing changes in the _super-repo_ "ls" to the script repo:

	```bash
	git pull ls main # Bring changes and merge all at once (recommended)
	```
	```bash
	git fetch ls main # Bring changes only, so then you can do "git diff ...ls/main" (or git diff ..ls/main file-name) to see changes before doing "git merge ls main"
	```

	> :warning: **Warning:** In any case, all possible conflicts will have to be resolved. To delete all unnecesary common files detected as _deleted by us_ at once, you can make use the following command:
	>```bash
	>git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {} # Or its alias: grdu
	>```

* 2.2\. Before uploading local changes to the remote, it's always advisable to make sure that the script repo is up to date:

	```bash
	git pull origin main # To bring any possible changes from the script repo on GitHub
	```

	```bash
	git pull ls main # To bring any possible changes in common/shared files from an up-to-date local super-repo "ls"
	```

* 2.3\. Now, local changes can be pushed safely to the remote repository of the script on GitHub as usually:

	```bash
	git push origin main
	```

	> :memo: **Note:** When the time comes to create a release, finish off the `docs/index.htm` file in order to `README.md` is up to date (since it is actually a symlink) and everything ready for [Pages deployment](#4-pages-deployment).

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
<br>

---

### 4\. Pages deployment

* 4.1\. Any time GitHub Pages need to be updated and assuming a dummy `gh-pages` branch already exists (otherwise create it with: `git branch gh-pages`), do:

	```bash
	git checkout gh-pages
	git reset --hard main # Ensure gh-pages is equal to main, where the real changes should be, to ease things along
	git push origin gh-pages -f # Force the just-reset-branch push to remote
	git checkout main # Return to main branch (or whichever you were) to avoid start working on gh-pages by mistake!
	```

	Alternatively, one can just use alias `deploy` and confirm. In any case, all this ensures deployments are only made when necessary and without having to worry about possible merge conflicts and such... By design, ___index.htm___ is actually the script's README, but you can always create a next-to-it ___index.html___ to get rid of this dependency.

	> :warning: **Warning:** That's all the `gh-pages` branch is for and no work should ever be done on it or it will be irremediably lost!

<br>

[^1]: Current development should be done in _dev_ branch or derivates and merged with _main_ once significant progress has been made and tested. But more on this later...

[^2]: Based on settings suggested by the [ENVIRONMENT.md](./ENVIRONMENT.md) file. But more on this later too...

[^3]: `-b main` specifies the branch to clone (if omitted, the default one is cloned), `ls` is the name of repository's remote (or, if not applicable, its path) and, finally, `ls_my_script` is the name of the directory where the repository will be cloned.

[1]: <https://github.com/lost-scripts/ls> 'Go to "ls" super-repository on GitHub'
