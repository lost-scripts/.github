<h1 align="center">Environment</h1><br>

## 📄 .bashrc
```sh
#######################
# Git Command Aliases #
#######################

alias g='git' # Streamlines the use of commands and .gitconfig aliases (e.g. 'g lrai' and 'glrai')
alias ga='git a'
alias gaa='git aa'
alias gaas='git aas'
alias gaaa='git aaa'
alias gaaas='git aaas'
alias gac='git ac'
alias gacam='git acam'
alias gaac='git aac'
alias garc='git arc'
alias gb='git b'
alias gtype='git type'
alias gdump='git dump'
alias gci='git ci'
alias gcl='git cl'
alias gco='git co'
alias gcp='git cp'
#alias gdeploy='git deploy'
alias gdc='git dc'
alias gdprev='git dprev'
alias gdmd='git dmd'
alias gdt='git dt'
alias gdif='git dif'
alias gdir='git dir'
alias gdbur='git dbur'
alias gdbtr='git dbtr'
alias gdbua='git dbua'
alias gdbta='git dbta'
alias gft='git ft'
alias gftdif='git ftdif'
alias gfup='git fup'
alias glg='git lg'
alias gld='git ld'
alias glhist="git lhist"
alias glrai="git lrai"
alias glprev='git lprev'
alias glad='git lad'
alias glall="git lall"
alias gm2m='git m2m'
alias gm2d='git m2d'
alias gmt='git mt'
alias gmv='git mv'
alias grm='git rm'
alias grmc='git rmc'
alias gr='git r'
alias grs='git rs'
alias gpullauh='git pullauh'
alias grepod='git repod'
#alias grepoud='unset GIT_DIR' # Use function 'reposud' instead
alias grepow='git repow'
#alias grepouw='unset GIT_WORK_TREE' # Use function 'reposuw' instead
alias grepodw='git repodw'
alias gsh='git sh'
alias gs='git s'
alias gsu='git su'
alias gsuno='git suno'
alias gsbtr='git sbtr'
alias gsw='git sw'
alias gta='git ta'
alias gtr='git tr'
alias gwipe='git wipe'


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

function stpush() { # Push changes TO the repository of the theme
	local subtree_dir="${1:-themes/lost-theme}" # Use the provided argument as the subtree directory, or default to "themes/lost-theme"
	local bare_repo="../$(basename $subtree_dir)" # Compute the path to the sibling repo (assumes ../<subtree_name>)
	git subtree push --prefix="$subtree_dir" "$bare_repo" main && echo "Subtree push to '$bare_repo' completed!" || echo "Subtree push to '$bare_repo' failed!"
}

function stpull() { # Pull changes FROM the repository of the theme
	local subtree_dir="${1:-themes/lost-theme}"	# Use the provided argument as the subtree directory, or default to "themes/lost-theme"
	local bare_repo="../$(basename $subtree_dir)" # Compute the path to the sibling repo (assumes ../<subtree_name>)
	git subtree pull --prefix="$subtree_dir" "$bare_repo" main --squash && echo "Subtree pull to '$bare_repo' completed!" || echo "Subtree pull to '$bare_repo' failed!"
}


############
# Other... #
############

function reposud() { # (path) | Set/Unset GIT_DIR to a specific directory
	if [[ -z "$1" ]]; then
		unset GIT_DIR
		echo "GIT_DIR: UNSET! ($(git rev-parse --absolute-git-dir))"
	else
		local old=${GIT_DIR:-$(git rev-parse --absolute-git-dir)}
		export GIT_DIR="$1"
		echo "GIT_DIR: $old --> $GIT_DIR"
	fi
}

function reposuw() { # (path) | Set/Unset GIT_WORK_TREE to a specific directory
	if [[ -z "$1" ]]; then
		unset GIT_WORK_TREE
		echo "GIT_WORK_TREE: UNSET! ($(git rev-parse --show-toplevel))"
	else
		local old=${GIT_WORK_TREE:-$(git rev-parse --show-toplevel)}
		export GIT_WORK_TREE="$1"
		echo "GIT_WORK_TREE: $old --> $GIT_WORK_TREE"
	fi
}

function reposu() { # (path) | Set/Unset GIT_DIR & GIT_WORK_TREE for a specific repository
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

export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows perpetually (commented due to it causes tab problems afterwards!)


############
#   Hugo   #
############

alias h='hugo --gc --cleanDestinationDir --destination docs'
alias hs='hugo server --disableFastRender --destination public'


##############
# Bash Notes #
##############

# Check alias: type -a <command>
# Remember to reload .bashrc if you make any changes here either by alias "rel" or: source ~/.bashrc or . ~/.bashrc
```

## 📄 .gitconfig
```sh
[core]
	editor = \"C:\\Program Files\\Microsoft VS Code\\bin\\code\" --wait
	autocrlf = false
	eol = lf 
	symlinks = true
	excludesfile = C:\\Users\\Rai\\.gitignore
[user]
	email = rai.lopez@outlook.com
	name = Rai
[init]
	defaultBranch = main
[merge "ours"]
	driver = true
[alias]
	a = add
	aa = add .
	aas = add . && git status -u
	aaa = add --all
	aaas = "!git add --all && git status -u"
	ac = "!git add . && git commit -m"
	acam = "!git add . && git commit --amend"
	aac = "!git add -A && git commit -m"
	arc = "!f() { mkdir -p _releases && git archive -o _releases/$(basename \"$(git rev-parse --show-toplevel)\").zip main; }; f"
	b = branch
	type = cat-file -t
	dump = cat-file -p
	ci = commit -m
	cl = clone
	co = checkout
	cp = cherry-pick
	#deploy ='!git checkout pages && git reset --hard main && git push origin pages -f' # Use the function instead!
	dc = diff --cached
	dprev = diff main..origin/main # Preview diffs of changes
	dmd = diff main..dev # Preview branches diff
	dt = difftool
	dif = diff --word-diff
	dir = git rev-parse --git-dir
	dbur = "!git status --porcelain | grep 'DU' | cut -c 4- | xargs -I {} git rm {}"   # A1. Resolve REMOVING all "deleted by us"
	dbtr = "!git status --porcelain | grep 'DT' | cut -c 4- | xargs -I {} git rm {}"   # A2. Resolve REMOVING all "deleted by them"
	dbua = "!git status --porcelain | grep '^DU' | cut -c 4- | xargs -I {} git add {}" # B1. Resolve ADDING all "deleted by us"
	dbta = "!git status --porcelain | grep '^DT' | cut -c 4- | xargs -I {} git add {}" # B2. Resolve ADDING all "deleted by them"
	ft = fetch
	ftdif = "!git fetch && git diff HEAD..@{u}" # Only see the changes that would come with git pull
	fup = fetch origin --prune # Fetch/Update/Prune (trim dead branches)
	lg = log --graph --pretty=oneline --abbrev-commit --decorate # Nice log output
	ld = log --graph --full-history --date-order --all --date=format:'%Y%m%d-%H%M' --pretty=format:'%x08%x09 %C(green)%h%C(reset) %C(cyan)AD%C(reset)·%C(magenta)CD%C(reset): %C(cyan)%ad%C(reset)·%C(magenta)%cd%C(reset) %C(cyan)%an%C(reset)·%C(magenta)%cn%C(reset)%C(red)%d%C(reset) %s' # Add e.g. ` -n 8` to limit output
	lhist = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
	lrai = log --graph --full-history --date-order --date=format:'%Y%m%d-%H%M' --pretty=format:'%x08%x09 %C(green)%h%C(reset) %C(cyan)%ad%C(reset)%C(red)%d%C(reset) %s [%C(yellow)%aN%C(reset)]' # Add --all to include unreachable items
	lprev = log main..origin/main # Preview commit logs of changes
	lad = log --all --decorate --oneline --graph
	lall = log --pretty=format: --name-only --diff-filter=A | sort - | sed '/^$/d'
	m2m = "!f() { current=$(git branch --show-current); echo '[!] Syncing dev -> main...'; git checkout main && git merge dev && git checkout \"$current\"; }; f" # Merge 'dev' into 'main' and return to current branch
	m2d = "!f() { echo '[!] Syncing main -> dev...'; git merge main; }; f" # Merge 'main' into 'dev' (staying in dev)
	mt = mergetool
	mv = "mv "
	rm = "rm "
	rmc = "rm --cached "
	r = remote -vv
	rs = remote set-url # Remote new-url
	pullauh = git pull --allow-unrelated-histories # Merges histories of two projects that started their lives independently
	repod = "rev-parse --absolute-git-dir"
	#repoud = "unset GIT_DIR" # Use function 'reposud' instead
	repow = "rev-parse --show-toplevel"
	#repouw = "unset GIT_WORK_TREE" # Use function 'reposuw' instead
	repodw = "!git rev-parse --absolute-git-dir && git rev-parse --show-toplevel"
	sh = show --no-patch # Show tag info without diff
	s = status
	su = status -u
	suno = git status -uno
	sbtr = "!git log --all --format=%b | awk '/git-subtree-dir/{ print $NF }' | sort -u"
	sw = "switch "
	ta = tag -a # Add annotated tag: ta <tagname> [<hash> if not in commit] [-m <mensaje> to avoid editor]
	tr = "!git push origin :refs/tags/" # Remove remote tag: tr <tagname>
	wipe = "!f() { echo '[!] Wiping worktree in 6, 5, 4... (Ctrl+C to cancel)'; sleep 6; git reset --hard HEAD && git clean -f$1; }; f" # Restore worktree to match current commit (add ` d` to include dirs if necessary)
[diff]
	tool = vscode
[difftool]
	prompt = false
[difftool "vscode"]
	cmd = "code --diff \"$LOCAL\" \"$REMOTE\""
[merge]
	tool = vscode
[mergetool]
	prompt = false
[mergetool "vscode"]
	cmd = "code --wait $MERGED"
[color]
	ui = true
	branch = auto
	diff = auto
	status = auto
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

> :memo: **Note:** Hooks live in the central ".github" repository under "/hooks". Copy them to your repos and ensure they are activated: `git config core.hooksPath .github/hooks` and executable: `chmod +x .github/hooks/*` or `chmod +x <filepath>` (the flag is tracked by Git but may need to be restored on Windows). Local modifications are discouraged unless strictly necessary.

<dl><dd>

### 📄 hooks/pre-commit

- Hook script for reliably update *.lua files ScriptBuild variable
</dd></dl>

<dl><dd>

### 📄 hooks/post-commit

- Hook script for writing reliable ScriptBuild based *.lua.log files
</dd></dl>