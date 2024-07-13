<h1 align="center">Environment</h1><br>

## .bashrc
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

function gsd() { # Set/Unset directory
	if [[ -z "$1" ]]; then
		unset GIT_DIR
		echo "GIT_DIR: UNSET! ($(git rev-parse --absolute-git-dir))"
	else
		local old=${GIT_DIR:-$(git rev-parse --absolute-git-dir)}
		export GIT_DIR="$1"
		echo "GIT_DIR: $old --> $GIT_DIR"
	fi
}

function gswt() { # Set/Unset work tree
	if [[ -z "$1" ]]; then
		unset GIT_WORK_TREE
		echo "GIT_WORK_TREE: UNSET! ($(git rev-parse --show-toplevel))"
	else
		local old=${GIT_WORK_TREE:-$(git rev-parse --show-toplevel)}
		export GIT_WORK_TREE="$1"
		echo "GIT_WORK_TREE: $old --> $GIT_WORK_TREE"
	fi
}

function repo() {
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

#function post_worktree() { cd "$1" && ./.git/hooks/post-worktree.sh }


############
# Other... #
############

#export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows perpetually (commented due to it causes tab problems afterwards)


##############
# Bash Notes #
##############

# Remember to reload .bashrc if you make any changes here either by alias "rb" or: source ~/.bashrc or . ~/.bashrc
```

## .gitconfig
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

## .gitignore
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

## hooks

<dl><dd>

### pre-commit

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

### post-commit

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