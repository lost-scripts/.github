<h1 align="center">Environment</h1><br>

## 📄 .bashrc

```sh
#################
# Configuration #
#################

export PROJECTS="C:/Users/Rai/Projects" # See about using `%CSIDL_PROFILE%` or equivalent? E.g. "%CSIDL_PROFILE%/Projects"
export PROJECT="$PROJECTS/LS" # Main project container
export PROJECT_ALT="$PROJECTS/RL" # Aternative project container (e.g. for other/personal repos like lost-theme?)
export MSYS=winsymlinks:nativestrict # Allow symlink creation on Windows perpetually (comment if it causes tab problems afterwards)


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
alias gm2main='git m2main'
alias gm2dev='git m2dev'
alias gmt='git mt'
alias gmv='git mv'
alias grm='git rm'
alias grmc='git rmc'
alias gr='git r'
alias grs='git rs'
alias gpullauh='git pullauh'
alias gpom='git pom'
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

function stpush() { # Push changes TO the repository of the theme
	local subtree_dir="${1:-themes/lost-theme}" # Use the provided argument as the subtree directory, or default to "themes/lost-theme"
	local theme_name=$(basename "$subtree_dir") # Extract the folder name to use it as the repository name
	local bare_repo="${PROJECT_ALT}/$theme_name" # Compute the path to the specified or sibling (../<subtree_name>) repo
	if [[ ! -d "$bare_repo" ]]; then bare_repo="../$theme_name"; fi # Fallback to sibling directory if not found in PROJECT_ALT
	git subtree push --prefix="$subtree_dir" "$bare_repo" main && echo "✅ Subtree push to '$bare_repo' completed!" || echo "❌ Subtree push failed!"
}

function stpull() { # Pull changes FROM the repository of the theme
	local subtree_dir="${1:-themes/lost-theme}" # Use the provided argument as the subtree directory, or default to "themes/lost-theme"
	local theme_name=$(basename "$subtree_dir") # Extract the folder name to use it as the repository name
	local bare_repo="${PROJECT_ALT}/$theme_name" # Compute the path to the specified or sibling (../<subtree_name>) repo
	if [[ ! -d "$bare_repo" ]]; then bare_repo="../$theme_name"; fi # Fallback to sibling directory if not found in PROJECT_ALT
	git subtree pull --prefix="$subtree_dir" "$bare_repo" main --squash && echo "✅ Subtree pull from '$bare_repo' completed!" || echo "❌ Subtree pull failed!"
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
	excludesfile = ~/.gitignore
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
	lall = "!git log --pretty=format: --name-only --diff-filter=A | sort - | sed '/^$/d'"
	m2main = "!f() { current=$(git branch --show-current); echo \"[!] Fast-forwarding main to dev...\"; git checkout main && git merge dev --ff-only && git checkout \"$current\"; }; f" # Merge 'dev' into 'main' cleanly (ff) and return to current branch
	m2dev = "!f() { echo \"[!] Merging main into dev...\"; git merge main; }; f" # Merge 'main' into 'dev' (staying in dev)
	mt = mergetool
	mv = "mv "
	rm = "rm "
	rmc = "rm --cached "
	r = remote -vv
	rs = remote set-url # Remote new-url
	pullauh = git pull --allow-unrelated-histories # Merges histories of two projects that started their lives independently
	pom = push origin main
	repod="rev-parse --absolute-git-dir"
	#repoud="unset GIT_DIR" # Use function 'reposud' instead
	repow="rev-parse --show-toplevel"
	#repouw="unset GIT_WORK_TREE" # Use function 'reposuw' instead
	repodw="!git rev-parse --absolute-git-dir && git rev-parse --show-toplevel"
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
[credential]
	useHttpPath = false
[include]
	path = ~/.config/git/config
```

## 📂 .ssh/
```bash
├───📄 config               # Multi-account mapping & identity routing (See below)
├───📄 id_lost-scripts      # Main/Org Private Key: Lost-Scripts (DO NOT SHARE!)
├───📄 id_lost-scripts.pub  # Main/Org Public Key: Lost-Scripts (Uploaded to GitHub)
├───📄 id_railopez          # Personal Private Key, e.g. (DO NOT SHARE!)
├───📄 id_railopez.pub      # Personal Public Key, e.g. (Uploaded to GitHub)
└───📄 known_hosts          # Fingerprints of trusted servers (GitHub, etc.)
```

### 📄 config

```sh
# Main/Org Account (Name: lost-scripts | Use: git remote add origin git@github.com:lost-scripts/repo.git)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_lost-scripts

# Personal Account (e.g., Name: RaiLopez | Use: git remote add origin git@github-railopez:RaiLopez/repo.git)
Host github-railopez
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_railopez # e.g., ~/.ssh/id_railopez
```


## 📂 hooks (DEPRECATED)

<dl><dd>

### 📄 hooks/pre-commit (**DISCONTINUED**, use `BUILD_AUTOASSIST=true` in `.config` now)

- Hook script for reliably update *.lua files ScriptBuild variable 
</dd></dl>

<dl><dd>

### 📄 hooks/post-commit (**DISCONTINUED**)

- Hook script for writing reliable ScriptBuild based *.lua.log files
</dd></dl>

> 📝 **Note:** ~~Hooks lived in the central ".github" repository under "/hooks". Copy them to your repos and ensure they are activated: `git config core.hooksPath .github/hooks` and executable: `chmod +x .github/hooks/*` or `chmod +x <filepath>` (the flag is tracked by Git but may need to be restored on Windows). Local modifications are discouraged unless strictly necessary.~~