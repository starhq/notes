# Vscode配置:
## settings.json
~~~json
{
    // ─────────────────────────────
    // 🔤 Editor and UI
    // ─────────────────────────────
    "editor.fontFamily": "'Source Code Pro', 'JetBrains Mono', Menlo, Monaco, 'Courier New', monospace",
    "editor.fontSize": 16,
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true,
    "editor.formatOnType": true,
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
    "go.goroot": "~/.asdf/installs/golang/1.25.5/go",
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
    "java.jdt.ls.vmargs": "-XX:+UseParallelGC -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -Dsun.zip.disableMemoryMapping=true -Xmx4G -Xms512m -Xlog:disable",
    "java.configuration.maven.userSettings": "~/.asdf/installs/maven/3.9.12/conf/settings.xml",
    "java.import.gradle.home": "~/.asdf/installs/gradle/9.2.1",
    "java.import.gradle.wrapper.enabled": false,
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
        }
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
        }
    },

    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.codeActionsOnSave": {
            "source.fixAll.eslint": "explicit"
        }
    },

    "[typescriptreact]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.codeActionsOnSave": {
            "source.fixAll.eslint": "explicit"
        }
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
        }
    },

    // "[prisma]": {
    //     "editor.defaultFormatter": "Prisma.prisma"
    // },

    // ─────────────────────────────
    // 📄 File Handling
    // ─────────────────────────────
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "files.autoGuessEncoding": true,
    "files.associations": {
    "*.vue": "vue"
    },

    // ─────────────────────────────
    // ⚡ Code Runner
    // ─────────────────────────────
    "code-runner.runInTerminal": true,
    "code-runner.saveAllFilesBeforeRun": true,
    "code-runner.saveFileBeforeRun": true,

    // ─────────────────────────────
    // 🐳 Docker & YAML
    // ─────────────────────────────
    "terminal.integrated.defaultProfile.osx": "zsh",

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
    // ⚙️ Other Settings
    // ─────────────────────────────
    // "application.extensionMarketUrl": "https://marketplace.visualstudio.com/",
    // "vsintellicode.modify.editor.suggestSelection": "choseToUpdateConfiguration",
    "json.schemaDownload.enable": false,
    // "trae.tab.enableSmartEdit": false


}~~~