---
date: 2026-01-24
tags:
  - homebrew
  - system-config
  - git-config
  - iterm2
  - terminal-config
  - develop-env-config
  - docker-config
---

# 安装homebrew

```shell
# 1. 准备工作：安装 Xcode 命令行工具
if ! xcode-select -p &>/dev/null; then
  echo "正在安装 Xcode 命令行工具..."
  xcode-select --install
fi

# 2. 设置 Homebrew 镜像环境变量（注意：无尾部空格！）
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"

# 3. 将镜像配置永久写入 .zprofile (检查去重)
MARKER="# Homebrew Mirror Config"
if ! grep -q "$MARKER" ~/.zprofile; then
  cat >> ~/.zprofile << EOF
  
$MARKER
export HOMEBREW_BREW_GIT_REMOTE="$HOMEBREW_BREW_GIT_REMOTE"
export HOMEBREW_API_DOMAIN="$HOMEBREW_API_DOMAIN"
export HOMEBREW_BOTTLE_DOMAIN="$HOMEBREW_BOTTLE_DOMAIN"
EOF
fi

# 4. 使用中科大提供的安装脚本安装 Homebrew
if ! command -v brew &>/dev/null; then
  echo "正在安装 Homebrew..."
  /bin/bash -c "$(curl -fsSL https://mirrors.ustc.edu.cn/misc/brew-install.sh)"
    # 官网
# /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
else
  echo "✅ Homebrew 已安装"
fi

# 5. 动态检测并配置 Homebrew 运行路径 (兼容 M系列和Intel)
UNAME_MACHINE=$(uname -m)
if [[ "$UNAME_MACHINE" == "arm64" ]]; then
    BREW_PATH="/opt/homebrew/bin/brew"
else
    BREW_PATH="/usr/local/bin/brew"
fi

# 6. 配置 Shell 环境 (检查去重)
if [[ -x "$BREW_PATH" ]]; then
  if ! grep -q "brew shellenv" ~/.zprofile; then
    echo "eval \"\$($BREW_PATH shellenv)\"" >> ~/.zprofile
  fi
  eval "$($BREW_PATH shellenv)"
fi

# 7. 刷新配置并测试
brew update
brew doctor || true
```

# 安装常用软件

```shell
cat > /tmp/Brewfile << 'EOF'

# CLI 工具
brew "eza"
brew "fzf"
brew "fd"
brew "ripgrep"
brew "bat"
brew "jq"
brew "sd"
brew "zoxide"
brew "entr"
brew "asdf"
brew "ffmpeg"
brew "lazydocker"
brew "dive"
brew "ctop"
brew "qwen-code"

# GUI 应用
cask "intellij-idea"
cask "clash-verge-rev"
cask "google-chrome"
cask "obsidian"
cask "raycast"
cask "warp"
cask "the-unarchiver"
cask "iterm2"
cask "postman" # API开发和测试工具 
cask "docker-desktop" # 容器化平台 
cask "wechat" # 微信
cask "baidunetdisk" # 百度网盘
# cask "microsoft-remote-desktop" # 远程桌面
# cask "proxyman" # 抓包
# cask "termius" # ssh
# cask "imageoptim" # 图片压缩工具
# cask "cursor" # 基于VSCode的AI辅助编码IDE 
# cask "discord" # 语音、视频和文字聊天应用 
# cask "whatsapp" # 跨平台即时通讯应用 
# cask "cleanshot" # 强大的屏幕截图和录屏工具 
# cask "screen-studio" # 高质量屏幕录制软件 
# cask "imageoptim" # 无损图片压缩工具 
# cask "bitwarden" # 开源密码管理器 
# cask "obs" # 开源直播和录屏软件 
# cask "elgato-stream-deck" # Stream Deck控制器的配套软件 
# cask "elgato-camera-hub" # Elgato相机控制软件 
# cask "zoom" # 视频会议软件 
# cask "vlc" # 开源多媒体播放器 
# cask "pgadmin4" # PostgreSQL数据库管理工具 
# cask "nordvpn" # VPN客户端 
# cask "zed" # 高性能代码编辑器 
# cask "ngrok" # 内网穿透工具，暴露本地服务器到公网

#字体
cask "font-source-code-pro"
EOF

brew bundle --file=/tmp/Brewfile
```

# 系统设置

## 终端设置
```shell
# Dock
defaults write com.apple.dock autohide -bool true
defaults write com.apple.dock autohide-delay -float 0
defaults write com.apple.dock autohide-time-modifier -float 0.3

# 禁止 .DS_Store
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

# 键盘重复速度
defaults write NSGlobalDomain KeyRepeat -int 1
defaults write NSGlobalDomain InitialKeyRepeat -int 10

# 触控板轻点点击
defaults write com.apple.AppleMultitouchTrackpad Clicking -bool true
defaults -currentHost write -g com.apple.mouse.tapBehavior -int 1

# Finder
defaults write com.apple.finder ShowPathbar -bool true
defaults write com.apple.finder AppleShowAllFiles -bool true          # 显示隐藏文件
defaults write com.apple.finder _FXSortFoldersFirst -bool true
defaults write com.apple.finder _FXSortFoldersFirstOnDesktop -bool true
defaults write NSGlobalDomain AppleShowAllExtensions -bool true        # 显示所有后缀

  # 其他
defaults write com.apple.menuextra.battery ShowPercent -bool true

killall Dock Finder
```

## 手动设置
```text
**触控板：** 系统设置 → 触控板 → 轻点来点按 ✅
**三指拖拽：** 辅助功能 → 指针控制 → 触控板选项 → 拖移样式
**键盘：** 修饰键 → Caps Lock改成Control（程序员必备）
```

# git设置

```shell
# Git 全局配置
git config --global user.name "starhq"
git config --global user.email "starimba@gmail.com"
# 推荐：防止不同平台换行符报错
git config --global core.autocrlf input
git config --global core.editor vim # 或 code、nano
git config --global init.defaultBranch main # 设置默认分支为main

# 创建 GitHub 专用密钥目录
mkdir -p ~/.ssh/github
chmod 700 ~/.ssh ~/.ssh/github

# 仅当密钥不存在时生成
KEY_PATH="$HOME/.ssh/github/id_ed25519"
if [ ! -f "$KEY_PATH" ]; then
  ssh-keygen -t ed25519 -C "starimba@gmail.com" -f "$KEY_PATH" -N ""
fi

# 修复权限
chmod 600 "$KEY_PATH"
chmod 644 "$KEY_PATH.pub"

# 配置 SSH 选项
SSH_CONFIG="$HOME/.ssh/config"
grep -Fxq "Host github.com" "$SSH_CONFIG" || cat >> "$SSH_CONFIG" << 'EOF'

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github/id_ed25519
    IdentitiesOnly yes
EOF
chmod 600 "$SSH_CONFIG"

# 复制公钥到剪贴板（仅在 macOS 上适用）
if command -v pbcopy &> /dev/null; then
    pbcopy < "$KEY_PATH.pub"
    echo "✅ SSH 密钥已生成并复制到剪贴板，请到 GitHub 添加："
    echo "   https://github.com/settings/ssh/new"
else
    echo "🔑 SSH 密钥已生成。请手动复制以下内容到 GitHub："
    cat "$KEY_PATH.pub"
    echo "👉 访问: https://github.com/settings/ssh/new"
fi
```

# iterm2设置

```shell
# 1. 下载 iTerm2 Material Design 主题
echo "🎨 Downloading iTerm2 Material Design theme..."
curl -Ls "https://gh-proxy.org/https://raw.githubusercontent.com/MartinSeeler/iterm2-material-design/master/material-design-colors.itermcolors" \
  -o /tmp/material-design-colors.itermcolors && open /tmp/material-design-colors.itermcolors

# 2. 安装 Oh My Zsh（如果未安装）
if [ ! -d "$HOME/.oh-my-zsh" ]; then
  echo "📦 Installing Oh My Zsh..."
  sh -c "$(curl -fsSL "https://gh-proxy.org/https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh")" "" --unattended
fi

# 3. 定义插件目录（统一使用 $HOME）
ZSH_CUSTOM_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"

# 4. 安装插件（如果未存在）
echo "🔌 Installing plugins..."

## 4.1 函数：安装插件（如果未存在）
install_plugin() {
  local plugin_name="$1"
  local plugin_repo="$2"
  local plugin_dir="$ZSH_CUSTOM_DIR/plugins/$plugin_name"
  
  if [ ! -d "$plugin_dir" ]; then
    echo "🔌 Installing $plugin_name..."
    git clone "https://gh-proxy.org/${plugin_repo}" "$plugin_dir"
  else
    echo "✅ $plugin_name already installed."
  fi
}

## 4.2 安装插件
### autosuggestions
install_plugin "zsh-autosuggestions" "https://github.com/zsh-users/zsh-autosuggestions.git"
### syntax highlighting
install_plugin "zsh-syntax-highlighting" "https://github.com/zsh-users/zsh-syntax-highlighting.git"
### history substring search
install_plugin "zsh-history-substring-search" "https://github.com/zsh-users/zsh-history-substring-search.git"

# 5. 自动更新 .zshrc（启用插件 + 绑定按键）
ZSHRC="$HOME/.zshrc"
# 如果未启用插件，则更新 plugins 行
if ! grep -q "zsh-autosuggestions" "$ZSHRC"; then
  sed -i.bak "s/plugins=(git)/plugins=(docker zsh-autosuggestions zsh-syntax-highlighting copypath copyfile copybuffer zsh-history-substring-search)/" "$ZSHRC"
  echo "✅ Plugins enabled in ~/.zshrc"
fi

echo
echo "🎉 Setup complete!"
echo "💡 Next steps:"
echo "  1. 重启终端 或 运行: source ~/.zshrc"
echo "  2. 在 iTerm2 中：Profiles → Colors → Color Presets → 导入 material-design-colors"
```


# 命令行工具配置

```shell
# ===================================================================
# 1. 配置 ~/.fzfrc（增强预览）
# ===================================================================
if [ ! -f ~/.fzf.zsh ]; then
  echo "⚙️  Configuring fzf shell integration..."
  "$(brew --prefix)/opt/fzf/install" --all --no-bash --no-fish
fi

cat > ~/.fzfrc << 'EOF'

export FZF_DEFAULT_COMMAND='fd --type f --strip-cwd-prefix --hidden --follow --exclude .git'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"

export FZF_DEFAULT_OPTS="--height 60% --layout=reverse --border --inline-info \
  --color='header:italic' \
  --preview 'bat --color=always --style=numbers --line-range :500 {} 2>/dev/null || cat {}' \
  --preview-window right:50%:wrap"
EOF

# ===================================================================
# 2. 配置 ~/.zshrc（健壮 + 兼容）
# ===================================================================
ZSHRC="$HOME/.zshrc"

# 3.1 确保 HOMEBREW_PREFIX 有默认值（防空）
if ! grep -q "HOMEBREW_PREFIX" "$ZSHRC"; then
  sed -i.bak "1i\\
: \${HOMEBREW_PREFIX:=\"/opt/homebrew\"}
" "$ZSHRC"
fi

# 3.2 添加主配置块（只添加一次）
if ! grep -q "SHELL FRAMEWORKS & PLUGINS" "$ZSHRC"; then
  cat >> "$ZSHRC" << 'EOF'
  
setopt CORRECT_ALL

# ===================================================================
# 🔧 SHELL FRAMEWORKS & PLUGINS
# ===================================================================

: ${HOMEBREW_PREFIX:="/opt/homebrew"}

# fzf 配置
[ -f $HOME/.fzf.zsh ] && source $HOME/.fzf.zsh
[ -f $HOME/.fzfrc ] && source $HOME/.fzfrc

# zoxide
eval "$(zoxide init zsh --cmd z)"
# z (简化 cd)
alias cd="z"

# asdf 配置
[ -f "$HOMEBREW_PREFIX/opt/asdf/libexec/asdf.sh" ] && source "$HOMEBREW_PREFIX/opt/asdf/libexec/asdf.sh"
# java_home 配置
[ -f ~/.asdf/plugins/java/set-java-home.zsh ] && source ~/.asdf/plugins/java/set-java-home.zsh

# 自动添加 bindkey（如果不存在）
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down
bindkey '^p' history-substring-search-up
bindkey '^n' history-substring-search-down
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-down

# ===================================================================
# 🖥️ ALIASES
# ===================================================================

# eza (现代 ls)
alias ls="eza --icons --time-style=long-iso"
alias ll="ls -lh"
alias la="ll -a"
alias lg="la --git"
alias tree="eza -T"

# bat (现代 cat)
alias cat="bat"

# tldr (简化 man)
alias man="tldr"

# 快速清理
alias docker-clean='docker system prune -af'
alias docker-rm-stopped='docker rm $(docker ps -aq --filter status=exited)'

# 快速进入容器
alias dexec='docker exec -it'

# 查看容器日志（带时间戳）
alias dlog='docker logs -f --timestamps'

# ===================================================================
# 🔑 SSH UTILITIES
# ===================================================================

server() {
  local hosts
  hosts=$(grep -E '^Host [^*[:space:]]+' ~/.ssh/config 2>/dev/null | cut -d' ' -f2 | sort)
  [ -z "$hosts" ] && { echo "⚠️ No SSH hosts in ~/.ssh/config"; return 1; }
  local selected
  selected=$(echo "$hosts" | fzf --prompt="ssh> " --height=40% --reverse)
  [ -n "$selected" ] && ssh "$selected"
}

keys() {
  if [ $# -eq 0 ]; then
    echo "Usage: keys <server_ip> [user]"
    echo "Example: keys 192.168.1.1 root"
    return 1
  fi

  local SERVER_IP="$1"
  local SSH_USER="${2:-$(whoami)}"
  local KEY_DIR="$HOME/.ssh/$SERVER_IP"
  local CONFIG_FILE="$HOME/.ssh/config"
  local MARKER="### MANAGED: $SERVER_IP ###"

  mkdir -p "$KEY_DIR" || { echo "❌ Failed to create key dir"; return 1; }

  if [ ! -f "$KEY_DIR/id_ed25519" ]; then
    echo "🔑 Generating key for $SERVER_IP..."
    ssh-keygen -t ed25519 -C "$SSH_USER@$SERVER_IP" -f "$KEY_DIR/id_ed25519" -N "" || { echo "❌ Key generation failed"; return 1; }
  else
    echo "✅ Key already exists: $KEY_DIR/id_ed25519"
  fi

  chmod 700 "$HOME/.ssh" "$KEY_DIR"
  chmod 600 "$KEY_DIR/id_ed25519"
  chmod 644 "$KEY_DIR/id_ed25519.pub"

  [ -f "$CONFIG_FILE" ] || touch "$CONFIG_FILE"
  chmod 600 "$CONFIG_FILE"
  cp "$CONFIG_FILE" "$CONFIG_FILE.bak" 2>/dev/null

  # 安全替换配置（修复 macOS sed 问题）
  local ESCAPED_MARKER
  ESCAPED_MARKER=$(printf '%s\n' "$MARKER" | sed 's/[[\.*^$()+?{|]/\\&/g')
  if [[ "$OSTYPE" == "darwin"* ]]; then
    sed -i '' -e "/^${ESCAPED_MARKER}/,/^$/d" -e "/^${ESCAPED_MARKER}/,\$d" "$CONFIG_FILE" 2>/dev/null
  else
    sed -i.bak -e "/^${ESCAPED_MARKER}/,/^$/d" -e "/^${ESCAPED_MARKER}/,\$d" "$CONFIG_FILE" 2>/dev/null
  fi

  cat >> "$CONFIG_FILE" << EOF
$MARKER
Host $SERVER_IP
    HostName $SERVER_IP
    User $SSH_USER
    IdentityFile $KEY_DIR/id_ed25519
    IdentitiesOnly yes
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null

EOF

  echo "📝 ~/.ssh/config updated for $SERVER_IP"
  echo "💡 Run: ssh-copy-id -i $KEY_DIR/id_ed25519.pub $SSH_USER@$SERVER_IP"
}

# ===================================================================
# ⚙️ OTHERS
# ===================================================================

cfg() {
  local file
  file=$(find ~/.asdf -type f \( -name "*.properties" -o -name "*.xml" -o -name "gradle.properties" -o -name "mavenrc" -o -name ".mavenrc" \) 2>/dev/null | fzf)
  [ -n "$file" ] && vi "$file"
}

clash() {
  cd ~/workspace/clash/  && ./update-clash-sources.sh 
}

gradle-mirror() {
  local MIRROR_URL="https\\://mirrors.cloud.tencent.com/gradle"
 
  # 解析参数
  case "$1" in
    aliyun)
      MIRROR_URL="https\\://mirrors.aliyun.com/gradle"
      ;;
    huawei)
      MIRROR_URL="https\\://repo.huaweicloud.com/gradle"
      ;;
    "")
      # 默认腾讯云
      ;;
    *)
      ;;
  esac

  # 检查是否在 Gradle 项目根目录
  if [[ ! -f "gradle/wrapper/gradle-wrapper.properties" ]]; then
    echo "❌ 当前目录不是 Gradle 项目根目录（未找到 gradle/wrapper/gradle-wrapper.properties）"
    return 1
  fi

  # 检查是否已安装 sd
  if ! command -v sd &> /dev/null; then
    echo "❌ 未找到 'sd' 命令，请先安装：brew install sd"
    return 1
  fi

  # 执行替换
  echo "🔄 正在将 Gradle 镜像源替换为: ${MIRROR_URL//\\/}"
  sd 'https\\://services.gradle.org/distributions' "$MIRROR_URL" gradle/wrapper/gradle-wrapper.properties

  if [[ $? -eq 0 ]]; then
    echo "✅ 替换成功！新配置:"
    grep "distributionUrl" gradle/wrapper/gradle-wrapper.properties
  else
    echo "❌ 替换失败，请检查文件权限或内容格式"
    return 1
  fi
}

# The following lines have been added by Docker Desktop to enable Docker CLI completions.
fpath=($HOME/.docker/completions $fpath)
autoload -Uz compinit
compinit
# End of Docker CLI completions

# GRADLE_USER_HOME
export GRADLE_USER_HOME="$HOME/.gradle"

# commitizen-practice-config
echo '{"path": "cz-conventional-changelog"}' > ~/.czrc 
EOF
fi
```

# 开发环境配置

## node.js

```shell
asdf plugin add nodejs https://gh-proxy.org/https://github.com/asdf-vm/asdf-nodejs.git

# 注意lts版本,可能还要重启终端
asdf install nodejs 24.12.0
asdf global nodejs 24.12.0

npm config set registry https://registry.npmmirror.com
corepack enable

npm install -g typescript tldr commitizen cz-conventional-changelog release-it
```

## java

```shell
asdf plugin add java https://gh-proxy.org/https://github.com/halcyon/asdf-java.git

asdf install java temurin-25.0.1+8.0.LTS
asdf install global temurin-25.0.1+8.0.LTS
```

## maven

```shell
asdf plugin add maven https://gh-proxy.org/https://github.com/halcyon/asdf-maven.git

asdf install maven 3.9.12
asdf global maven 3.9.12
```

## rust
~~~Shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~~

bun
~~~Shell
curl -fsSL https://bun.com/install | bash
~~~

### 以下信息需要写入maven的setting.xml文件中
```xml
<!-- mirror节点 -->
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>central,jcenter,google,spring,gradle-plugin</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
 
<mirror>
  <id>huaweicloud</id>
  <mirrorOf>*</mirrorOf>
  <name>华为云镜像</name>
  <url>https://repo.huaweicloud.com/repository/maven/</url>
</mirror>

<!-- profile 节点 -->
<profile>
  <id>jdk-25</id>
  <properties>
    <maven.compiler.source>25</maven.compiler.source>
    <maven.compiler.target>25</maven.compiler.target>
    <maven.compiler.release>25</maven.compiler.release>
  </properties>
</profile>
<profile>
  <id>parallel-downloads</id>
  <properties>
    <maven.artifact.threads>5</maven.artifact.threads>
  </properties>
</profile>

<!-- 激活profile -->
<activeProfiles>
  <activeProfile>jdk-25</activeProfile>
  <activeProfile>parallel-downloads</activeProfile>
</activeProfiles>
```


## gradle

```shell
asdf plugin add gradle https://gh-proxy.org/https://github.com/rfrancis/asdf-gradle.git

asdf install gradle 9.2.1
asdf global gradle 9.2.1

mkdir -p ~/.gradle/init.d
cat > ~/.gradle/init.d/init.gradle << 'EOF'
allprojects {
    // 1. 国内镜像源
    repositories {
		mavenLocal()                                                   
        maven { url 'https://maven.aliyun.com/repository/public/' }
		maven { url 'https://maven.aliyun.com/repository/jcenter/'}
        maven { url 'https://maven.aliyun.com/repository/google/' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
        mavenCentral()
		gradlePluginPortal()
    }
    
    // 2. 全局配置
    configurations.all {
        // 动态版本缓存时间（默认24小时，可缩短）
        resolutionStrategy.cacheDynamicVersionsFor 10, 'minutes'
        // 更改模块缓存时间
        resolutionStrategy.cacheChangingModulesFor 4, 'hours'
    }
}
EOF


cat >> ~/.gradle/gradle.properties << 'EOF'
# 并行构建（核心优化）
org.gradle.parallel=true
# 配置on-demand（按需）模式，只构建相关项目
org.gradle.configureondemand=true
# 启用构建缓存
org.gradle.caching=true
# 设置JVM堆大小
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=1g -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
# 启用守护进程（加速后续构建）
org.gradle.daemon=true
# 日志级别（quiet减少输出）
org.gradle.logging.level=quiet
EOF
```

## golang

```shell
asdf plugin add golang https://gh-proxy.org/https://github.com/asdf-community/asdf-golang.git

asdf install golang 1.25.5
asdf global golang 1.25.5

echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zprofile
source ~/.zprofile
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GO111MODULE=on

asdf current >> ~/.tool-versions 2>/dev/null
```

# docker

```shell
softwareupdate --install-rosetta

cat >> ~/.docker/daemon.json << EOF
  
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "registry-mirrors": [
    "https://n1eudwtr.mirror.aliyuncs.com",
    "https://docker.xuanyuan.me"
  ]
}
EOF
```

# 完整脚本

```shell
#!/bin/zsh

# ========================================================================
# macOS 一键初始化脚本（适用于 Apple Silicon & Intel Mac）
# 作者：基于你的原始配置优化整合
# 日期：2026-01-01
# ========================================================================

set -euo pipefail

echo "🚀 开始 macOS 环境初始化..."

# ========================================================================
# 1. 安装 Homebrew（使用清华镜像）
# ========================================================================
install_homebrew() {
  echo "📦 安装/配置 Homebrew..."

  # 安装 Xcode 命令行工具
  if ! xcode-select -p &>/dev/null; then
    echo "正在安装 Xcode 命令行工具..."
    xcode-select --install
    echo "请在弹窗中完成安装后，按回车继续..."
    read
  fi

  # 设置清华镜像（最推荐）
  export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
  export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
  export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"

  # 永久写入 .zprofile
  MARKER="# Homebrew Mirror Config (TUNA)"
  if ! grep -q "$MARKER" ~/.zprofile; then
    cat >> ~/.zprofile << EOF

$MARKER
export HOMEBREW_BREW_GIT_REMOTE="$HOMEBREW_BREW_GIT_REMOTE"
export HOMEBREW_BOTTLE_DOMAIN="$HOMEBREW_BOTTLE_DOMAIN"
export HOMEBREW_API_DOMAIN="$HOMEBREW_API_DOMAIN"
EOF
    echo "✅ Homebrew 镜像已写入 ~/.zprofile"
  fi

  # 安装 Homebrew（如果未安装）
  if ! command -v brew &>/dev/null; then
    echo "正在安装 Homebrew..."
    /bin/bash -c "$(curl -fsSL https://mirrors.ustc.edu.cn/misc/brew-install.sh)"
  else
    echo "✅ Homebrew 已安装"
  fi

  # 动态检测路径
  UNAME_MACHINE=$(uname -m)
  if [[ "$UNAME_MACHINE" == "arm64" ]]; then
    BREW_PATH="/opt/homebrew/bin/brew"
  else
    BREW_PATH="/usr/local/bin/brew"
  fi

  # 添加到 shell 环境
  if [[ -x "$BREW_PATH" ]]; then
    if ! grep -q "brew shellenv" ~/.zprofile; then
      echo "eval \"\$($BREW_PATH shellenv)\"" >> ~/.zprofile
    fi
    eval "$($BREW_PATH shellenv)"
  fi

  brew update
  brew doctor || true
}

# ========================================================================
# 2. 安装常用软件（Brewfile）
# ========================================================================
install_software() {
  echo "📲 安装常用软件..."

  cat > /tmp/Brewfile << 'EOF'
# CLI 工具
brew "eza"
brew "fzf"
brew "fd"
brew "ripgrep"
brew "bat"
brew "jq"
brew "sd"
brew "zoxide"
brew "entr"
brew "asdf"
brew "ffmpeg"
brew "lazydocker"
brew "dive"
brew "ctop"
brew "qwen-code"

# GUI 应用
cast "intellij-idea"
cask "clash-verge-rev"
cask "google-chrome"
cask "obsidian"
cask "raycast"
cask "warp"
cask "the-unarchiver"
cask "iterm2"
cask "wechat" # 微信
cask "baidunetdisk" # 百度网盘
cask "postman" # API开发和测试工具
cask "docker-desktop" # 容器化平台 
# cask "microsoft-remote-desktop" # 远程桌面
# cask "proxyman" # 抓包
# cask "termius" # ssh
# cask "imageoptim" # 图片压缩工具
# cask "cursor" # 基于VSCode的AI辅助编码IDE 
# cask "discord" # 语音、视频和文字聊天应用 
# cask "whatsapp" # 跨平台即时通讯应用 
# cask "cleanshot" # 强大的屏幕截图和录屏工具  
# cask "screen-studio" # 高质量屏幕录制软件 
# cask "imageoptim" # 无损图片压缩工具 
# cask "bitwarden" # 开源密码管理器 
# cask "obs" # 开源直播和录屏软件 
# cask "elgato-stream-deck" # Stream Deck控制器的配套软件 
# cask "elgato-camera-hub" # Elgato相机控制软件 
# cask "zoom" # 视频会议软件 
# cask "vlc" # 开源多媒体播放器 
# cask "pgadmin4" # PostgreSQL数据库管理工具 
# cask "nordvpn" # VPN客户端 
# cask "zed" # 高性能代码编辑器 
# cask "ngrok" # 内网穿透工具，暴露本地服务器到公网

# 字体
cask "font-source-code-pro"
# cask "font-hack-nerd-font"           # Hack字体的Nerd Font版本，适合编程
# cask "font-menlo-for-powerline"      # 针对Powerline优化的Menlo字体
# cask "font-jetbrains-mono"           # JetBrains开发的等宽编程字体
# cask "font-jetbrains-mono-nerd-font" # JetBrains Mono的Nerd Font版本
EOF

  brew bundle --file=/tmp/Brewfile
}

# ========================================================================
# 3. 系统偏好设置
# ========================================================================
system_preferences() {
  echo "⚙️  配置系统偏好..."

  # Dock
  defaults write com.apple.dock autohide -bool true
  defaults write com.apple.dock autohide-delay -float 0
  defaults write com.apple.dock autohide-time-modifier -float 0.3

  # 禁止 .DS_Store
  defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
  defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

  # 键盘重复速度
  defaults write NSGlobalDomain KeyRepeat -int 1
  defaults write NSGlobalDomain InitialKeyRepeat -int 10

  # 触控板轻点点击
  defaults write com.apple.AppleMultitouchTrackpad Clicking -bool true
  defaults -currentHost write -g com.apple.mouse.tapBehavior -int 1

  # Finder
  defaults write com.apple.finder ShowPathbar -bool true
  defaults write com.apple.finder AppleShowAllFiles -bool true          # 显示隐藏文件
  defaults write com.apple.finder _FXSortFoldersFirst -bool true
  defaults write com.apple.finder _FXSortFoldersFirstOnDesktop -bool true
  defaults write NSGlobalDomain AppleShowAllExtensions -bool true        # 显示所有后缀

  # 其他
  defaults write com.apple.menuextra.battery ShowPercent -bool true

  killall Dock Finder
}

# ========================================================================
# 4. Git & SSH 配置
# ========================================================================
configure_git_ssh() {
  echo "🔧 配置 Git 与 SSH..."

  git config --global user.name "starhq"
  git config --global user.email "starimba@gmail.com"
  git config --global core.autocrlf input
  git config --global core.editor vim # 或 code、nano
  git config --global init.defaultBranch main # 设置默认分支为main

  mkdir -p ~/.ssh/github
  chmod 700 ~/.ssh ~/.ssh/github

  KEY_PATH="$HOME/.ssh/github/id_ed25519"
  if [ ! -f "$KEY_PATH" ]; then
    ssh-keygen -t ed25519 -C "starimba@gmail.com" -f "$KEY_PATH" -N ""
  fi

  chmod 600 "$KEY_PATH"
  chmod 644 "$KEY_PATH.pub"

  SSH_CONFIG="$HOME/.ssh/config"
  if ! grep -Fxq "Host github.com" "$SSH_CONFIG" 2>/dev/null; then
    cat >> "$SSH_CONFIG" << 'EOF'

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github/id_ed25519
    IdentitiesOnly yes
EOF
    chmod 600 "$SSH_CONFIG"
  fi

  if command -v pbcopy &>/dev/null; then
    pbcopy < "$KEY_PATH.pub"
    echo "✅ SSH 公钥已复制到剪贴板，请前往 https://github.com/settings/ssh/new 添加"
  fi
}

# ========================================================================
# 5. iTerm2 + Oh My Zsh + 插件
# ========================================================================
configure_iterm_ohmyzsh() {
  echo "🎨 配置 iTerm2 主题与 Oh My Zsh..."

  # 现代 Material 主题（活跃仓库）
  curl -Ls https://raw.githubusercontent.com/mbadolato/iTerm2-Color-Schemes/master/schemes/MaterialDesignColors.itermcolors \
    -o /tmp/material.itermcolors && open /tmp/material.itermcolors

  # Oh My Zsh
  if [ ! -d "$HOME/.oh-my-zsh" ]; then
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
  fi

  ZSH_CUSTOM_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"

  install_plugin() {
    local name="$1" repo="$2"
    local dir="$ZSH_CUSTOM_DIR/plugins/$name"
    if [ ! -d "$dir" ]; then
      git clone "https://github.com/$repo" "$dir"
    fi
  }

  install_plugin "zsh-autosuggestions"      "zsh-users/zsh-autosuggestions.git"
  install_plugin "zsh-syntax-highlighting"  "zsh-users/zsh-syntax-highlighting.git"
  install_plugin "zsh-history-substring-search" "zsh-users/zsh-history-substring-search.git"

  # 启用插件
  if ! grep -q "zsh-autosuggestions" ~/.zshrc; then
    sed -i.bak 's/plugins=(git)/plugins=(docker zsh-autosuggestions zsh-syntax-highlighting zsh-history-substring-search copypath copyfile copybuffer)/' ~/.zshrc
  fi
}

# ========================================================================
# 6. 命令行工具增强配置（fzf + 别名 + 函数）
# ========================================================================
configure_shell_tools() {
  echo "🔧 配置 fzf、别名与自定义函数..."

  # fzf
  if [ ! -f ~/.fzf.zsh ]; then
    "$(brew --prefix)/opt/fzf/install" --all --no-bash --no-fish
  fi

  cat > ~/.fzfrc << 'EOF'
export FZF_DEFAULT_COMMAND='fd --type f --strip-cwd-prefix --hidden --follow --exclude .git'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_DEFAULT_OPTS="--height 60% --layout=reverse --border --inline-info \
  --color='header:italic' \
  --preview 'bat --color=always --style=numbers --line-range :500 {} 2>/dev/null || cat {}' \
  --preview-window right:50%:wrap"
EOF

  # 主配置块写入 .zshrc
  if ! grep -q "SHELL FRAMEWORKS & PLUGINS" ~/.zshrc; then
    cat >> ~/.zshrc << 'EOF'
    
setopt CORRECT_ALL

# ===================================================================
# 🔧 SHELL FRAMEWORKS & PLUGINS
# ===================================================================

: ${HOMEBREW_PREFIX:="/opt/homebrew"}

[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
[ -f ~/.fzfrc ] && source ~/.fzfrc

eval "$(zoxide init zsh --cmd z)"
alias cd="z"

[ -f "$HOMEBREW_PREFIX/opt/asdf/libexec/asdf.sh" ] && source "$HOMEBREW_PREFIX/opt/asdf/libexec/asdf.sh"
[ -f ~/.asdf/plugins/java/set-java-home.zsh ] && source ~/.asdf/plugins/java/set-java-home.zsh

# History substring search keybindings
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down
bindkey '^p' history-substring-search-up
bindkey '^n' history-substring-search-down

# ===================================================================
# 🖥️ ALIASES
# ===================================================================
alias ls="eza --icons --time-style=long-iso"
alias ll="ls -lh"
alias la="ll -a"
alias lg="la --git"
alias tree="eza -T"
alias cat="bat"
alias man="tldr"

# 快速清理
alias docker-clean='docker system prune -af'
alias docker-rm-stopped='docker rm $(docker ps -aq --filter status=exited)'

# 快速进入容器
alias dexec='docker exec -it'

# 查看容器日志（带时间戳）
alias dlog='docker logs -f --timestamps'

# ===================================================================
# 🔑 SSH 快捷函数（server / keys）
# ===================================================================
# （这里可粘贴你原来的 server() 和 keys() 函数，篇幅原因略）
server() {
  local hosts
  hosts=$(grep -E '^Host [^*[:space:]]+' ~/.ssh/config 2>/dev/null | cut -d' ' -f2 | sort)
  [ -z "$hosts" ] && { echo "⚠️ No SSH hosts in ~/.ssh/config"; return 1; }
  local selected
  selected=$(echo "$hosts" | fzf --prompt="ssh> " --height=40% --reverse)
  [ -n "$selected" ] && ssh "$selected"
}

keys() {
  if [ $# -eq 0 ]; then
    echo "Usage: keys <server_ip> [user]"
    echo "Example: keys 192.168.1.1 root"
    return 1
  fi

  local SERVER_IP="$1"
  local SSH_USER="${2:-$(whoami)}"
  local KEY_DIR="$HOME/.ssh/$SERVER_IP"
  local CONFIG_FILE="$HOME/.ssh/config"
  local MARKER="### MANAGED: $SERVER_IP ###"

  mkdir -p "$KEY_DIR" || { echo "❌ Failed to create key dir"; return 1; }

  if [ ! -f "$KEY_DIR/id_ed25519" ]; then
    echo "🔑 Generating key for $SERVER_IP..."
    ssh-keygen -t ed25519 -C "$SSH_USER@$SERVER_IP" -f "$KEY_DIR/id_ed25519" -N "" || { echo "❌ Key generation failed"; return 1; }
  else
    echo "✅ Key already exists: $KEY_DIR/id_ed25519"
  fi

  chmod 700 "$HOME/.ssh" "$KEY_DIR"
  chmod 600 "$KEY_DIR/id_ed25519"
  chmod 644 "$KEY_DIR/id_ed25519.pub"

  [ -f "$CONFIG_FILE" ] || touch "$CONFIG_FILE"
  chmod 600 "$CONFIG_FILE"
  cp "$CONFIG_FILE" "$CONFIG_FILE.bak" 2>/dev/null

  # 安全替换配置（修复 macOS sed 问题）
  local ESCAPED_MARKER
  ESCAPED_MARKER=$(printf '%s\n' "$MARKER" | sed 's/[[\.*^$()+?{|]/\\&/g')
  if [[ "$OSTYPE" == "darwin"* ]]; then
    sed -i '' -e "/^${ESCAPED_MARKER}/,/^$/d" -e "/^${ESCAPED_MARKER}/,\$d" "$CONFIG_FILE" 2>/dev/null
  else
    sed -i.bak -e "/^${ESCAPED_MARKER}/,/^$/d" -e "/^${ESCAPED_MARKER}/,\$d" "$CONFIG_FILE" 2>/dev/null
  fi

  cat >> "$CONFIG_FILE" << EOF
$MARKER
Host $SERVER_IP
    HostName $SERVER_IP
    User $SSH_USER
    IdentityFile $KEY_DIR/id_ed25519
    IdentitiesOnly yes
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null

EOF

  echo "📝 ~/.ssh/config updated for $SERVER_IP"
  echo "💡 Run: ssh-copy-id -i $KEY_DIR/id_ed25519.pub $SSH_USER@$SERVER_IP"
}

# ===================================================================
# ⚙️ BUILD TOOLS
# ===================================================================

cfg() {
  local file
  file=$(find ~/.asdf -type f \( -name "*.properties" -o -name "*.xml" -o -name "gradle.properties" -o -name "mavenrc" -o -name ".mavenrc" \) 2>/dev/null | fzf)
  [ -n "$file" ] && vi "$file"
}

clash() {
  cd ~/workspace/clash/  && ./update-clash-sources.sh 
}

gradle-mirror() {
  local MIRROR_URL="https\\://mirrors.cloud.tencent.com/gradle"
 
  # 解析参数
  case "$1" in
    aliyun)
      MIRROR_URL="https\\://mirrors.aliyun.com/gradle"
      ;;
    huawei)
      MIRROR_URL="https\\://repo.huaweicloud.com/gradle"
      ;;
    "")
      # 默认腾讯云
      ;;
    *)
      ;;
  esac

  # 检查是否在 Gradle 项目根目录
  if [[ ! -f "gradle/wrapper/gradle-wrapper.properties" ]]; then
    echo "❌ 当前目录不是 Gradle 项目根目录（未找到 gradle/wrapper/gradle-wrapper.properties）"
    return 1
  fi

  # 检查是否已安装 sd
  if ! command -v sd &> /dev/null; then
    echo "❌ 未找到 'sd' 命令，请先安装：brew install sd"
    return 1
  fi

  # 执行替换
  echo "🔄 正在将 Gradle 镜像源替换为: ${MIRROR_URL//\\/}"
  sd 'https\\://services.gradle.org/distributions' "$MIRROR_URL" gradle/wrapper/gradle-wrapper.properties

  if [[ $? -eq 0 ]]; then
    echo "✅ 替换成功！新配置:"
    grep "distributionUrl" gradle/wrapper/gradle-wrapper.properties
  else
    echo "❌ 替换失败，请检查文件权限或内容格式"
    return 1
  fi
}

# The following lines have been added by Docker Desktop to enable Docker CLI completions.
fpath=($HOME$/.docker/completions $fpath)
autoload -Uz compinit
compinit
# End of Docker CLI completions

export GRADLE_USER_HOME="$HOME/.gradle"

# commitizen-practice-config
echo '{"path": "cz-conventional-changelog"}' > ~/.czrc
EOF
  fi
}

# ========================================================================
# 7. 开发环境（asdf + Node/Java/Maven/Gradle/Go/Docker）
# ========================================================================
setup_dev_env() {
  echo "☕ 配置开发环境（asdf）..."

  asdf plugin add nodejs https://github.com/asdf-vm/asdf-nodejs.git || true
  asdf plugin add java https://github.com/halcyon/asdf-java.git || true
  asdf plugin add maven https://github.com/halcyon/asdf-maven.git || true
  asdf plugin add gradle https://github.com/rfrancis/asdf-gradle.git || true
  asdf plugin add golang https://github.com/asdf-community/asdf-golang.git || true

  # 2026 年最新稳定版
  asdf install nodejs latest:24
  asdf install java temurin-25.0.1+8.0.LTS   # Java 25 LTS
  asdf install maven 3.9.12
  asdf install gradle 9.2.1
  asdf install golang 1.25.5

  asdf global nodejs latest:24
  asdf global java temurin-25.0.1+8.0.LTS
  asdf global maven 3.9.12
  asdf global gradle 9.2.1
  asdf global golang 1.25.5

  npm config set registry https://registry.npmmirror.com
  corepack enable
  npm install -g typescript tldr commitizen cz-conventional-changelog release-it

  # Go 配置
  echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.zprofile
  go env -w GOPROXY=https://goproxy.cn,direct
  go env -w GO111MODULE=on
  
  softwareupdate --install-rosetta

  cat >> ~/.docker/daemon.json << EOF
  
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "registry-mirrors": [
    "https://n1eudwtr.mirror.aliyuncs.com",
    "https://docker.xuanyuan.me"
  ]
}
EOF
}

# ========================================================================
# 主流程
# ========================================================================
main() {
  install_homebrew
  install_software
  system_preferences
  configure_git_ssh
  configure_iterm_ohmyzsh
  configure_shell_tools
  setup_dev_env

  echo "🎉🎉🎉 所有配置已完成！"
  echo ""
  echo "请执行以下操作让配置生效："
  echo "   1. 重启终端（推荐）"
  echo "   或手动运行："
  echo "      source ~/.zshrc"
  echo "      source ~/.zprofile"
  echo ""
  echo "手动操作提醒："
  echo "   • iTerm2 → Profiles → Colors → 导入 material-design-colors"
  echo "   • 系统设置 → 触控板 → 轻点来点按"
  echo "   • 辅助功能 → 指针控制 → 触控板选项 → 三指拖拽"
  echo "   • 键盘 → 修饰键 → Caps Lock 改成 Control"
  echo "   • docker需要ghcr.io/sky22333/hubproxy:latest和portainer/portainer-ce:alpine"
  echo ""
  echo "Enjoy your new Mac! 🚀"
}

main
```