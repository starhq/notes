# Vscode配置:

## settings.json
~~~json
{
  // ─────────────────────────────
  // 🔤 Editor and UI
  // ─────────────────────────────
  "editor.fontFamily": "'Source Code Pro', 'JetBrains Mono', Menlo, Monaco, 'Courier New', monospace",
  "editor.fontSize": 16,
  "editor.lineHeight": 1.5,
  "editor.letterSpacing": 0.5,
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.formatOnType": true,
  "editor.minimap.enabled": true,
  "editor.wordWrap": "on",
  "editor.cursorSmoothCaretAnimation": "on",
  "editor.cursorBlinking": "smooth",
  "editor.smoothScrolling": true,
  "editor.mouseWheelZoom": true,
  "editor.acceptSuggestionOnEnter": "smart",
  "editor.suggest.snippetsPreventQuickSuggestions": false,
  "editor.suggestSelection": "first",
  "editor.suggest.insertMode": "replace",

  "workbench.list.smoothScrolling": true,
  "workbench.editor.enablePreview": false,
  "workbench.iconTheme": "material-icon-theme",

  "window.dialogStyle": "custom",
  "window.zoomLevel": 1,

  // ─────────────────────────────
  // 🐹 Go (Golang)
  // ─────────────────────────────
  "go.goroot": "~/.asdf/installs/golang/1.25.5",
  "go.gopath": "~/go",
  "go.coverOnTestPackage": true,
  "go.useLanguageServer": true,
  "go.toolsManagement.autoUpdate": true,
  "go.lintTool": "staticcheck",
  "go.lintFlags": ["--severity", "warning"],
  "go.formatTool": "goimports",
  "go.delveConfig": {
    "debugAdapter": "dlv-dap",
    "showGlobalVariables": true
  },

  "gopls": {
    "build.buildFlags": ["-tags=wireinject"],
    "staticcheck": true,
    "completeUnimported": true,
    "usePlaceholders": true,
    "completionDocumentation": true,
    "hoverKind": "SynopsisDocumentation"
  },

  "[go]": {
    "editor.defaultFormatter": "golang.go",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    },
    "editor.formatOnSave": true
  },

  // ─────────────────────────────
  // ☕ Java
  // ─────────────────────────────
  "java.jdt.ls.java.home": "~/.asdf/installs/java/temurin-25.0.1+8.0.LTS",
  "java.jdt.ls.vmargs": "-XX:+UseParallelGC -Xmx2G -Xms512m",
  "java.configuration.maven.userSettings": "~/.asdf/installs/maven/3.9.12/conf/settings.xml",
  "java.import.gradle.wrapper.enabled": true,
  "java.configuration.runtimes": [
    {
      "name": "JavaSE-25",
      "path": "~/.asdf/installs/java/temurin-25.0.1+8.0.LTS",
      "default": true
    }
  ],
  "java.debug.settings.hotCodeReplace": "auto",
  "java.errors.incompleteClasspath.severity": "ignore",
  "java.format.enabled": true,
  "java.saveActions.organizeImports": true,
  "java.completion.matchCase": "off",
  "java.autobuild.enabled": true,

  "[java]": {
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    },
    "editor.formatOnSave": true
  },

  // ─────────────────────────────
  // 🌐 Web Development (JavaScript/TypeScript)
  // ─────────────────────────────
  "javascript.suggest.completeFunctionCalls": true,
  "typescript.suggest.completeFunctionCalls": true,
  "typescript.updateImportsOnFileMove.enabled": "always",

  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    },
    "editor.formatOnSave": true
  },

  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    },
    "editor.formatOnSave": true
  },

  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    },
    "editor.formatOnSave": true
  },

  // ─────────────────────────────
  // 🐍 PYTHON
  // ─────────────────────────────
  "[python]": {
    "editor.insertSpaces": true,
    "editor.tabSize": 4,
    "editor.formatOnSave": true
  },

  // ─────────────────────────────
  // 📝 MARKDOWN
  // ─────────────────────────────
  "[markdown]": {
    "editor.wordWrap": "on"
  },

  // ─────────────────────────────
  // 🎨 CSS & HTML Support
  // ─────────────────────────────
  "html-css-class-completion.includeGlobPattern": "**/*.{css,scss,less}",
  "html-css-class-completion.enableEmmetSupport": true,

  "emmet.includeLanguages": {
    "javascript": "javascriptreact"
  },

  // ─────────────────────────────
  // 🧪 Linting & Formatting
  // ─────────────────────────────
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue"
  ],

  "prettier.enable": true,

  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": "explicit"
    },
    "editor.formatOnSave": true
  },

  // ─────────────────────────────
  // 📄 File Handling
  // ─────────────────────────────
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000,
  "files.hotExit": "onExit",
  "files.encoding": "utf8",
  "files.autoGuessEncoding": true,
  "files.associations": {
    "*.vue": "vue"
  },

   // "[prisma]": {
    //     "editor.defaultFormatter": "Prisma.prisma"
    // },

  // ─────────────────────────────
  // 💡 edit suggest
  // ─────────────────────────────
  "editor.snippetSuggestions": "top",
  "editor.suggest.showKeywords": true,
  "editor.suggest.preview": true,
  "editor.suggest.shareSuggestSelections": true,
  "editor.inlineSuggest.enabled": true,

  // ─────────────────────────────
  // ⚡ Code Runner
  // ─────────────────────────────
  "code-runner.runInTerminal": true,
  "code-runner.saveAllFilesBeforeRun": true,
  "code-runner.saveFileBeforeRun": true,

  // ─────────────────────────────
  // 🐳 Docker & YAML
  // ─────────────────────────────
  "[dockercompose]": {
    "editor.insertSpaces": true,
    "editor.tabSize": 2,
    "editor.autoIndent": "advanced",
    "editor.defaultFormatter": "redhat.vscode-yaml"
  },

  "[github-actions-workflow]": {
    "editor.defaultFormatter": "redhat.vscode-yaml"
  },

  // ─────────────────────────────
  // 💻 Terminal Configuration
  // ─────────────────────────────
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.defaultProfile.osx": "zsh",
  "terminal.integrated.fontFamily": "'Source Code Pro', 'JetBrains Mono', monospace",
  "terminal.integrated.fontSize": 14,

  // ─────────────────────────────
  // ⚙️ Other Settings
  // ─────────────────────────────
  "json.schemaDownload.enable": false,
  "cursor.cpp.enablePartialAccepts": true,

  // optimize project index
  "search.exclude": {
    "**/node_modules": true,
    "**/bower_components": true,
    "**/*.code-search": true,
    "**/target": true,
    "**/.gradle": true,
    "**/build": true,
    "**/dist": true,
    "**/out": true
  },
  "files.watcherExclude": {
    "**/.git/objects/**": true,
    "**/node_modules/**": true,
    "**/target/**": true,
    "**/.gradle/**": true
  }
}
~~~

# git 配置信息

## .gitconfig

~~~ini
[user]
	name = starhq
	email = starimba@gmail.com
[core]
	excludesfile = ~/.gitignore_global
[difftool "sourcetree"]
	cmd = opendiff \"$LOCAL\" \"$REMOTE\"
	path = 
[mergetool "sourcetree"]
	cmd = /Applications/Sourcetree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
	trustExitCode = true
[commit]
	template = ~/.stCommitMsg
[init]
	defaultBranch = main
[alias]
  	# 基础
  	st = status -sb
  	co = checkout
  	cob = checkout -b
  	br = branch
  	bra = branch -a
  	ci = commit
  	cm = commit -m
  	ca = commit --amend
  	caf = commit --amend --no-edit

  	# 分支管理
  	brd = branch -d
  	brD = branch -D
  	brc = "!f() { git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d; }; f"
  	feat = "!f() { git checkout -b feature/$1; }; f"
  	fix = "!f() { git checkout -b fix/$1; }; f"

  	# 日志
  	lg = log --oneline --graph --all
	ll = log --pretty=format:\"%C(yellow)%h%C(reset) %C(green)%ad%C(reset) %C(blue)[%an]%C(reset) %s\" --date=short
  	last = log -n

 	# 远程
 	pl = pull --rebase
 	fpr = fetch --prune
 	psu = push -u origin HEAD

 	# 修复
 	unstage = reset HEAD --
  	discard = checkout --
  	undo = reset --soft HEAD~1
  	rb = rebase -i HEAD~3
  	rbf = "!f() { git rebase -i --autosquash $(git merge-base HEAD main); }; f"

  	# 状态
  	untracked = ls-files --others --exclude-standard
  	sl = stash list
 	track = branch -vv

	# 交互式 checkout
  	fco = "!f() { git checkout $(git branch --all | fzf --height 40% --reverse | sed 's/.* //'); }; f"
	# 仅在特性分支上允许 rebase
  	safe-rebase = "!f() { if [[ $(git branch --show-current) != main && $(git branch --show-current) != develop ]]; then git rebase main; else echo 'Cannot rebase main/develop!'; fi; }; f"
~~~

## ..gitignore

~~~TEXT
HELP.md  
.gradle  
build/  
!gradle/wrapper/gradle-wrapper.jar  
!**/src/main/**/build/  
!**/src/test/**/build/  
  
### STS ###  
.apt_generated  
.classpath  
.factorypath  
.project  
.settings  
.springBeans  
.sts4-cache  
bin/  
!**/src/main/**/bin/  
!**/src/test/**/bin/  
  
### IntelliJ IDEA ###  
.idea  
*.iws  
*.iml  
*.ipr  
out/  
!**/src/main/**/out/  
!**/src/test/**/out/  
  
### NetBeans ###  
/nbproject/private/  
/nbbuild/  
/dist/  
/nbdist/  
/.nb-gradle/  
  
### VS Code ###  
.vscode/
~~~