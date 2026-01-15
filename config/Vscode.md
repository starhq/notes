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