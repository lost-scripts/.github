<h1 align="center">Environment</h1><br>

## 📄 .bashrc
```sh
#######################
# Git Command Aliases #
#######################

alias g='git'
alias ga='git add'
alias gaa='git add .'
alias gaas='git add . && git status -u'
alias gaaa='git add --all'
alias gaaas='git add --all && git status -u'
alias gac='git add . && git commit -m'
alias gaca='git add . && git commit --amend' # Not working as expected...
alias gaac='git add -A && git commit -m'
alias garc='mkdir -p _releases && git archive -o _releases/$(basename "$(git rev-parse --show-toplevel)").zip main'
alias gb='git branch'
alias gtype='cat-file -t'
alias gdumb='cat-file -p'
alias gco='git checkout'
alias gc='git commit -m'
#alias gdeploy='git checkout pages && git reset --hard main && git push origin pages -f' # Use function below instead
alias gdc='git diff --cached'
alias gdprev='git diff main..origin/main' # Preview diffs of changes
alias gdiff='git diff --word-diff'
alias gdir='git rev-parse --git-dir'
alias gd='git rev-parse --absolute-git-dir'
#alias gud='unset GIT_DIR' # Use function below instead
alias gw='git rev-parse --show-toplevel'
#alias guw='unset GIT_WORK_TREE' # Use function below instead
alias gdw='git rev-parse --absolute-git-dir && git rev-parse --show-toplevel'
alias gf='git fetch'
alias gfdiff='git fetch && git diff HEAD..@{u}' # Only see the changes that would come with git pull
alias gl='git log --graph --pretty=oneline --abbrev-commit --decorate # nice log output'
alias glhist="git log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short"
#alias glrai="git log --date-order --date=iso --graph --full-history --all --pretty=format:'%x08%x09%C(red)%h %C(cyan)%ai%x08%x08%x08%x08%x08%x08%x08%x08%x08%C(reset) %C(bold green)(%ar%x08%x08%x08%x08)%C(reset) %C(yellow)%aN:%C(reset)%C(bold yellow)%d %C(reset)%s'"
alias glrai="git log --date-order --date=format:'%Y%m%d-%H%M' --graph --full-history --all --pretty=format:'%x08%x09%C(cyan)%ad%C(reset) %C(green)%h%C(reset)%C(red)%d%C(reset) %s [%C(yellow)%aN%C(reset)]'"
alias glprev='git log main..origin/main' # Preview commit logs of changes
alias glad='git log --all --decorate --oneline --graph'
alias glall="git log --pretty=format: --name-only --diff-filter=A | sort - | sed '/^$/d'"
alias gmv='git mv '
alias grm='git rm '
alias grmc='git rm --cached '
alias gpullauh='git pull --allow-unrelated-histories' # Merges histories of two projects that started their lives independently
alias gr='git remote -v'
alias grs='git remote set-url' # Remote new-url
alias gs='git status'
alias gsu='git status -u'
alias gsuno='git status -uno'
alias gsw='git switch '

alias grdu='git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {}' # Resolve REMOVING all "deleted by us"
alias grdt='git status --porcelain | grep 'DT' | cut -c 4- | xargs -I {} git rm {}' # Resolve REMOVING all "deleted by them"

alias gadu='git status --porcelain | grep '^DU' | cut -c 4- | xargs -I {} git add {}' # Resolve ADDING all "deleted by us"
alias gadt='git status --porcelain | grep '^DT' | cut -c 4- | xargs -I {} git add {}' # Resolve ADDING all "deleted by them"


################
# Bash Aliases #
################

alias cd..='cd ..'
alias ..='cd ..'
alias less='less -r'
alias ls='ls -F --color --show-control-chars'
alias ll='ls -l'
alias rel='source ~/.bashrc'


##################
# Bash Functions #
##################

function new() { # (repoName) | Create a new Lost Script...
	# Prompt for repo name if not provided
	if [ -z "$1" ]; then
		read -p "Please enter a name in the format 'ls_my_script' to continue: " repo_name
	else
		repo_name="$1"
	fi

	# Determine the path to use
	if [[ "$repo_name" == ../* ]]; then # Interpret as relative
		repo_path="$repo_name"
	elif [[ "$repo_name" == /* ]]; then # Interpret as absolute in a Unix format (e.g. `/d/Rai/Scripts/ls_my_script`)
		repo_path="$repo_name"
	else # Otherwise, use default location (one level up from cloned repo)
		repo_path="../$repo_name"
	fi

	# Confirm before proceeding
	while true; do
		read -p "Proceed with the creation of new script '$repo_path'? (y/n): " -n 1 -r yn
		echo # Move to a new line
		case $yn in
			[Yy]* ) break;; # Proceed
			[Nn]* ) echo "Process cancelled (no script was created)."; return 1;; # Cancel process
			* ) echo "Please answer y (yes) or n (no).";;
		esac
	done

	# Check if the path already exists and is not empty
	if [ -d "$repo_path" ] && [ "$(ls -A "$repo_path")" ]; then
		echo "Error: The destination path '$repo_path' already exists and is not an empty directory."
		return 1
	fi
	mkdir -p "$repo_path" || { echo "Error: Unable to create directory at $repo_path."; return 1; }	# Create the directory if it does not exist

	# Get info relative to the super-repo before leaving it!
	super_repo_remote=$(git remote get-url origin) # Extract the remote URL of the super-repo
	account=$(echo "$super_repo_remote" | sed -E 's|https?://[^/]+/([^/]+)/.*|\1|') # Extract account from the URL
	default_remote="https://github.com/$account/$repo_name.git" # Suggest a default remote URL based on account and repository name.

	# Start cloning and stuff...
	git clone . "$repo_path" -o ls # The `-o ls` is for directly rename the remote `origin` to `ls`
	cd "$repo_path" || exit 1 # Leave the super-repo and make clone the current directory!
	git remote set-url ls ../ls # Make path to the remote "ls" relative
	mv docs/templets/README.md docs/README.md # Start overwriting files which content differs from the ones in the super-repo...
	mv docs/templets/.gitattributes docs/.gitattributes
	mv docs/templets/.gitignore docs/.gitignore
	rmdir docs/templets 2>/dev/null # Try to silently remove the container folder if it is empty

	# Function to ask if a file should be kept
	ask_keep_file() {
		while true; do
			read -p "Do you want to keep: $1? (y/n): " -n 1 -r yn
			echo # Move to a new line
			case $yn in
				[Yy]* ) return 0;; # Keep file
				[Nn]* ) return 1;; # Do not keep file
				* ) echo "Please answer y (yes) or n (no).";;
			esac
		done
	}

	# Loop through and query each file in Modules and Utility folders
	for dir in Modules Utility; do
		if [ -d "$dir" ]; then
			for file in "$dir"/*; do
				if [ -f "$file" ]; then # Verify that it is a file
					if ask_keep_file "$file"; then
						: #echo "Keeping: $file" # The : is a no-op command to avoid empty block errors!
					else
						git rm -q "$file" # Remove file silently
					fi
				fi
			done
			rmdir "$dir" 2>/dev/null # Try to silently remove the container folder if it is empty
		fi
	done

	# Optionally, add --all and commit changes
	while true; do
		read -p "Do you want to commit the initial changes? (y/n): " -n 1 -r commit_yn
		echo # Move to a new line
		case $commit_yn in
			[Yy]* )
				git add -A
				git commit -m "$repo_name: Initial commit"
				echo "Initial commit done :D"
				break;;
			[Nn]* )
				echo "Initial commit skipped!"
				break;;
			* ) echo "Please answer y (yes) or n (no).";;
		esac
	done

	# Promp to a add the default remote, add a custom one or skip
	while true; do
		read -p "Add remote? (y (yes) adds default: '$default_remote'; enter new one adds that; n (no) skips; Enter key confims):" response
		case $response in
			[Yy]* ) remote_url=$default_remote; break;;
			[Nn]* ) remote_url=""; break;;
			* ) remote_url=$response; break;;
		esac
	done

	if [ -n "$remote_url" ]; then
		git remote add origin "$remote_url"
		git branch -M main # Just in case?
		#git push -u origin main # Maybe only if user answered y (yes) when prompted to commit? But this would assume the remote actually exists, so...
		echo "Remote 'origin' added with URL $remote_url"
	else
		echo "No remote URL added."
	fi

	echo "Repository ready at $repo_path (current directory)."
}

function deploy() { # GitHub Pages deployment using 'gh-pages' branch just for that...
	echo "WARNING: Branch 'gh-pages' will be overwritten locally and remotely!"
	read -p "Are you sure you want to commit all changes and deploy Pages? (y/n) " -n 1 -r
	echo # (optional) move to a new line
	if [[ $REPLY =~ ^[Yy]$ ]]
	then
		current_branch=$(git branch --show-current)
		#cp -f README.md docs/index.html # Copy & Rename Method (3 lines): Copy README.md and paste it as index.html in docs folder
		#git add -A # Stage all changes (instead of: git add docs/index.html?)
		#git commit -m "Predeployment commit: ensure main has index.html current contents" 
		git checkout gh-pages
		git reset --hard main
		git push origin gh-pages -f
		git checkout "$current_branch"
		echo "Deployment complete. Returned to branch '$current_branch'."
	else
		echo "Deployment canceled."
	fi
}

function updatedocs() {
	prev_msys=$MSYS
	export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows (if run as admin or in developer mode)
	rm -f docs; ln -s ScriptResources/"$1"/docs docs
	export MSYS=$prev_msys
}

function updatereadme() {
	prev_msys=$MSYS
	export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows (if run as admin or in developer mode)
	rm -f README.md; ln -s ScriptResources/"$1"/index.html README.md
	export MSYS=$prev_msys
}

function updateslsdocs() {
	prev_msys=$MSYS
	export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows (if run as admin or in developer mode)
	rm -rf ScriptResources/ls/docs; ln -s ../../../ls/ScriptResources/ls/docs ScriptResources/ls/docs
	export MSYS=$prev_msys
}

function updateall() {
	prev_msys=$MSYS
	export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows (if run as admin or in developer mode)
	rm -f docs; ln -s ScriptResources/"$1"/docs docs
	rm -f README.md; ln -s ScriptResources/"$1"/index.html README.md
	rm -rf ScriptResources/ls/docs; ln -s ../../../ls/ScriptResources/ls/docs ScriptResources/ls/docs
	export MSYS=$prev_msys
}


############
# Other... #
############

function gsd() { # (path) | Set/Unset GIT_DIR to a specific directory
	if [[ -z "$1" ]]; then
		unset GIT_DIR
		echo "GIT_DIR: UNSET! ($(git rev-parse --absolute-git-dir))"
	else
		local old=${GIT_DIR:-$(git rev-parse --absolute-git-dir)}
		export GIT_DIR="$1"
		echo "GIT_DIR: $old --> $GIT_DIR"
	fi
}

function gswt() { # (path) | Set/Unset GIT_WORK_TREE to a specific directory
	if [[ -z "$1" ]]; then
		unset GIT_WORK_TREE
		echo "GIT_WORK_TREE: UNSET! ($(git rev-parse --show-toplevel))"
	else
		local old=${GIT_WORK_TREE:-$(git rev-parse --show-toplevel)}
		export GIT_WORK_TREE="$1"
		echo "GIT_WORK_TREE: $old --> $GIT_WORK_TREE"
	fi
}

function repo() { # (path) | Set/Unset GIT_DIR & GIT_WORK_TREE for a specific repository
	if [ $# -gt 0 ]; then
			# Make absolute the path in the first parameter.
			path=`(cd "$1" ; pwd)`
			export GIT_DIR="$path"/.git
			export GIT_WORK_TREE=$PWD
	else
			unset GIT_DIR
			unset GIT_WORK_TREE
	fi
}

#function post_worktree() { cd "$1" && ./.git/hooks/post-worktree.sh }

#export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows perpetually (commented due to it causes tab problems afterwards!)


##############
# Bash Notes #
##############

# Remember to reload .bashrc if you make any changes here either by alias "rb" or: source ~/.bashrc or . ~/.bashrc
```

## 📄 .gitconfig
```sh
[core]
	editor = \"C:\\Users\\Ramon0\\AppData\\Local\\Programs\\Microsoft VS Code\\bin\\code\" --wait
	symlinks = true
	excludesfile = C:\\Users\\Ramon0\\.gitignore
[user]
	email = rai.lopez@outlook.com
	name = Rai
[alias]
	a = add
	aa = add .
	aaa = add --all
	ac = !git add . && git commit -m
	aac = !git add -A && git commit -m
	br = branch
	type = cat-file -t
	dump = cat-file -p
	co = checkout
	cp = cherry-pick
	cl = clone
	ci = commit
	#deploy ='git checkout pages && git reset --hard main && git push origin pages -f' # Use the function below instead
	dc = diff --cached
	dprev = diff main..origin/main # Preview diffs of changes
	diff = diff --word-diff
	ft = fetch
	ftdiff = !git fetch && git diff HEAD..@{u} # Only see the changes that would come with git pull
	lg = log --graph --pretty=oneline --abbrev-commit --decorate # Nice log output
	lhist = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
	#lrai = log --date-order --date=iso --graph --full-history --all --pretty=format:'%x08%x09%C(red)%h %C(cyan)%ai%x08%x08%x08%x08%x08%x08%x08%x08%x08%C(reset) %C(bold green)(%ar%x08%x08%x08%x08)%C(reset) %C(yellow)%aN:%C(reset)%C(bold yellow)%d %C(reset)%s'
	lrai = log --date-order --date=format:'%Y%m%d-%H%M' --graph --full-history --all --pretty=format:'%x08%x09%C(cyan)%ad%C(reset) %C(green)%h%C(reset)%C(red)%d%C(reset) %s [%C(yellow)%aN%C(reset)]'
	lprev = log main..origin/main # Preview commit logs of changes
	lad = log --all --decorate --oneline --graph
	lall = log --pretty=format: --name-only --diff-filter=A | sort - | sed '/^$/d'
	pullauh = git pull --allow-unrelated-histories # Merges histories of two projects that started their lives independently
	r = remote -v
	rs = remote set-url # Remote new-url
	st = status
	stu = status -u

	rdu = "!git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {}" # Resolve removing all "deleted by us"
	rdt = "!git status --porcelain | grep 'DT' | cut -c 4- | xargs -I {} git rm {}" # Resolve removing all "deleted by them"
[color]
	ui = true
	branch = auto
	diff = auto
	status = auto
[init]
	defaultBranch = main
[merge "ours"]
	driver = true
```

## 📄 .gitignore
```sh
#########
# PATHS #
#########

#Utility/*/


###########
# FOLDERS #
###########

_ARC/
_ARCH/
_BAK/
_BKP/
_BIN/
_DEV/
_DIS/
_OLD/
_OTH/
_OTHER/
_PRIV/
_PUB/
_RAW/
_REL/
_RES/
_RSC/
_RSCS/
_TEMP/
_TMP/
_TEST/
_TESTS/
_TRASH/
_TST/

_[Aa]rchive/
_[Bb]ackup/
_[Bb]in/
_[Dd]ev/
_[Dd]evelop/
_[Dd]evelopment/
_[Dd]isabled/
_[Oo]ther/
_[Rr]eleases/
_[Rr]esearch/
_[Rr]esources/
_[Tt]est/
_[Tt]ests/
_[Tt]rash/

#/ls_shapes_window/
#/ls_wiggle_window/
#ls_smart_manager/
#ls_smart_scripts/
#Modules/ls_modules.lua


##############
# FILE TYPES #
##############

*.dia~
*.log
*.bak
*.cache


#########
# FILES #
#########

_INFO.txt
_[Ii]nfo.txt
_NOTES.txt
_[Nn]otes.txt
_TEMP.txt
_[Tt]emp.txt
_TMP.txt
_[Tt]mp.txt
_TODO.txt
_[Tt]odo.txt
```

## 📂 hooks

<dl><dd>

### 📄 pre-commit

```sh
#!/bin/sh
# pre-commit hook script for reliably update *.lua files ScriptBuild variable

# Define the pattern to search for and the new build date
pattern="^ScriptBuild ="
new_build_date=$(date +"%Y%m%d-%H%M")

# Loop through all staged Lua files
git diff --cached --name-only | grep '\.lua$' | while IFS= read -r file; do
	# Check if the file contains the ScriptBuild line within the first 10 lines
	if head -n 10 "$file" | grep -m 1 -q "$pattern"; then
		# Determine the directory of the file
		file_dir=$(dirname "$file")

		# Find or create the backup directory
		backup_dir=$(find "$file_dir" -maxdepth 1 -type d \( -name '_arc*' -o -name '_archive' \) | head -n 1)
		if [ -z "$backup_dir" ]; then
			backup_dir="$file_dir/_archive"
			mkdir -p "$backup_dir" || backup_dir="$file_dir"
		fi

		# Update the ScriptBuild line and create a backup in the specified directory
		sed -i.bak "s/$pattern .*/ScriptBuild = \"$new_build_date\"/" "$file"
		# Convert LF to CRLF
		awk '{printf "%s\r\n", $0}' "$file" > "$file.tmp" && mv "$file.tmp" "$file"
		# Move the backup file to the backup directory
		mv "$file.bak" "$backup_dir/"

		# Validate, by simple check, the file integrity?
		#if ! grep -q "$pattern" "$file"; then
			#echo "Error: File integrity check failed for $file"
			#exit 1
		#fi

		# Add the modified file to the staging area
		git add "$file"
	fi
done
```
</dd></dl>

<dl><dd>

### 📄 post-commit

```sh
#!/bin/sh
# post-commit hook script for writing reliable ScriptBuild based *.lua.log files

# Define the pattern to search for
pattern="^ScriptBuild ="

# Loop through all committed Lua files
git diff-tree --no-commit-id --name-only -r HEAD | grep '\.lua$' | while IFS= read -r file; do
	# Check if the file contains the ScriptBuild line within the first 10 lines
	if head -n 10 "$file" | grep -m 1 -q "$pattern"; then
		# Get the commit message
		commit_message=$(git log -1 --pretty=%B)
		# Get the commit hash
		commit_hash=$(git log -1 --pretty=%h)
		# Get the modification date of the file
		mod_date=$(date -r "$file" +"%Y%m%d-%H%M")
		# Create or update the log file
		log_file="${file}.log"
		# Ensure the log file exists
		touch "$log_file"
		# Prepend the new entry to the log file
		echo "$mod_date ($commit_hash): $commit_message" | cat - "$log_file" > "${log_file}.tmp" && mv "${log_file}.tmp" "$log_file"
		# Mark the log file as hidden on Windows?
		#if [ "$(uname -o)" = "Msys" ]; then
			#attrib +h "$log_file"
		#fi
	fi
done
```
</dd></dl>