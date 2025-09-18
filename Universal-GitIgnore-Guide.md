# 通用 .gitignore 配置指南

本文档提供了一个全面的 `.gitignore` 配置模板，涵盖了常见项目类型和开发环境中应忽略的文件和目录。配置按类别组织，便于根据项目需求进行选择和定制。

## 📋 目录

- [操作系统文件](#操作系统文件)
- [开发工具配置](#开发工具配置)
- [AI 开发工具](#ai-开发工具)
- [依赖目录](#依赖目录)
- [构建产物](#构建产物)
- [日志文件](#日志文件)
- [环境变量文件](#环境变量文件)
- [测试相关文件](#测试相关文件)
- [缓存文件](#缓存文件)
- [临时文件](#临时文件)
- [数据库文件](#数据库文件)
- [媒体文件](#媒体文件)
- [压缩文件](#压缩文件)
- [完整配置模板](#完整配置模板)

---

## 操作系统文件

### macOS
```gitignore
# macOS 系统文件
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Icon?
```

### Windows
```gitignore
# Windows 系统文件
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.cab
*.msi
*.msix
*.msm
*.msp
*.lnk
```

### Linux
```gitignore
# Linux 系统文件
*~
.fuse_hidden*
.directory
.Trash-*
.nfs*
```

---

## 开发工具配置

### 传统 IDE 和编辑器
```gitignore
# Visual Studio Code
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
*.code-workspace

# JetBrains IDEs
.idea/
*.iws
*.iml
*.ipr
out/

# Sublime Text
*.sublime-project
*.sublime-workspace
.sublime-gulp.cache

# Atom
.atom/

# Vim
[._]*.s[a-v][a-z]
[._]*.sw[a-p]
[._]s[a-rt-v][a-z]
[._]ss[a-gi-z]
[._]sw[a-p]
Session.vim
Sessionx.vim
.netrwhist
*~
tags
[._]*.un~

# Emacs
*~
\#*\#
/.emacs.desktop
/.emacs.desktop.lock
*.elc
auto-save-list
tramp
.\#*

# Eclipse
.metadata
bin/
tmp/
*.tmp
*.bak
*.swp
*~.nib
local.properties
.settings/
.loadpath
.recommenders
.project
.classpath

# NetBeans
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

# Android Studio
*.iml
.gradle
/local.properties
/.idea/caches
/.idea/libraries
/.idea/modules.xml
/.idea/workspace.xml
/.idea/navEditor.xml
/.idea/assetWizardSettings.xml
.DS_Store
/build
/captures
.externalNativeBuild
.cxx
local.properties

# Xcode
*.pbxuser
!default.pbxuser
*.mode1v3
!default.mode1v3
*.mode2v3
!default.mode2v3
*.perspectivev3
!default.perspectivev3
xcuserdata/
*.moved-aside
*.xccheckout
*.xcscmblueprint
DerivedData/
*.hmap
*.ipa
*.xcuserstate
project.xcworkspace/
```

---

## AI 开发工具

### 主流 AI 编辑器
```gitignore
# Cursor (AI-powered code editor)
.cursor/
*.cursor-workspace
.cursor-tutor/

# GitHub Copilot
.copilot/
.github-copilot/

# Tabnine
.tabnine/
.tabnine-config/

# Codeium & Windsurf
.codeium/
codeium.json
.windsurf/
*.windsurf-workspace

# Replit (AI-powered online IDE)
.replit
replit.nix
.config/

# Sourcegraph Cody
.sourcegraph/
.cody/

# Amazon CodeWhisperer
.aws-codecatalyst/
.codecatalyst/

# JetBrains AI Assistant
.ai-assistant/
ai-assistant.xml

# Continue (VS Code AI extension)
.continue/
continue.json

# Aider (AI pair programming)
.aider*
aider.conf.yml

# Claude Dev / Anthropic
.claude/
.anthropic/

# OpenAI Codex related
.openai/
.codex/

# Zed (AI-enhanced editor)
.zed/
*.zed-workspace
```

### AWS Kiro (最新 AI IDE)
```gitignore
# Kiro AI IDE (AWS)
.kiro/
*.kiro-workspace
.kiro-specs/
kiro-config.json
*.kiro-session
.kiro-cache/
kiro-hooks/
.kiro-agents/
```

### 国内 AI 开发工具
```gitignore
# Trae AI IDE (字节跳动)
.trae/
.trae-ai/
trae-config.json
trae-workspace/
.trae-cache/

# 通义灵码 (阿里巴巴)
.tongyi/
.qwen/
.alibaba-ai/
tongyi-config.json
lingma-workspace/
.lingma/

# 豆包 MarsCode (字节跳动)
.marscode/
.doubao/
.bytedance-ai/
marscode-config.json
marscode-workspace/

# 文心一言 / 百度 Comate
.baidu-ai/
.comate/
.ernie/
comate-config.json
wenxin-workspace/

# 腾讯云 AI 代码助手 / CodeBuddy
.tencent-ai/
.coding-ai/
.qcloud-ai/
.codebuddy/
codebuddy-config.json

# 智谱 AI CodeGeeX
.codegeex/
.zhipu-ai/
.glm/
codegeex-config.json
codegeex-workspace/

# 华为云 CodeArts Snap
.huawei-ai/
.codearts/
.modelarts/
codearts-config.json

# 商汤 SenseCode
.sensecode/
.sensenova/
.sensetime-ai/

# 科大讯飞 iFlyCode
.iflycode/
.xfyun/
.iflytek-ai/

# 其他国产 AI 工具
.ai-china/
.domestic-ai/
ai-china-config/
*.cn-ai
```

### AI 通用配置
```gitignore
# AI chat histories and sessions
ai-chat-history/
.ai-sessions/
*.ai-chat
*.ai-session

# AI model caches
.ai-cache/
.model-cache/
ai-models/

# AI 代码生成工具
.ai-codegen/
.code-generator/
ai-generated/
*.ai-gen

# AI 重构工具
.ai-refactor/
.refactor-ai/
refactor-sessions/

# AI 测试生成
.ai-test/
.test-ai/
ai-test-gen/

# AI 文档生成
.ai-docs/
.doc-ai/
ai-documentation/

# AI 代码审查
.ai-review/
.review-ai/
ai-code-review/
```

---

## 依赖目录

### Node.js
```gitignore
# Node.js 依赖
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.pnpm-debug.log*

# Yarn
.yarn/*
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/sdks
!.yarn/versions
.pnp.*

# npm
.npm
.npmrc
package-lock.json
yarn.lock
pnpm-lock.yaml
```

### Python
```gitignore
# Python 依赖
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# Translations
*.mo
*.pot

# Django
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal

# Flask
instance/
.webassets-cache

# Scrapy
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
.python-version

# pipenv
Pipfile.lock

# PEP 582
__pypackages__/

# Celery
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/
```

### Java
```gitignore
# Java 依赖
*.class
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Gradle
.gradle
**/build/
!src/**/build/
gradle-app.setting
!gradle-wrapper.jar
!gradle-wrapper.properties
.gradletasknamecache
```

### PHP
```gitignore
# PHP 依赖
vendor/
composer.phar
composer.lock
.env.local
.env.*.local
```

### Ruby
```gitignore
# Ruby 依赖
*.gem
*.rbc
/.config
/coverage/
/InstalledFiles
/pkg/
/spec/reports/
/spec/examples.txt
/test/tmp/
/test/version_tmp/
/tmp/

# Used by dotenv library to load environment variables.
.env

# Ignore Byebug command history file.
.byebug_history

# Ignore bundler config.
/.bundle

# Ignore the default SQLite database.
/db/*.sqlite3
/db/*.sqlite3-journal
/db/*.sqlite3-*

# Ignore all logfiles and tempfiles.
/log/*
/tmp/*
!/log/.keep
!/tmp/.keep

# Ignore pidfiles, but keep the directory.
/tmp/pids/*
!/tmp/pids/
!/tmp/pids/.keep

# Ignore uploaded files in development.
/storage/*
!/storage/.keep

# Ignore master key for decrypting credentials and more.
/config/master.key

/public/assets
.byebug_history

# Ignore application configuration
/config/application.yml
```

### Go
```gitignore
# Go 依赖
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool, specifically when used with LiteIDE
*.out

# Dependency directories (remove the comment below to include it)
vendor/

# Go workspace file
go.work
```

### Rust
```gitignore
# Rust 依赖
# Generated by Cargo
# will have compiled files and executables
debug/
target/

# Remove Cargo.lock from gitignore if creating an executable, leave it for libraries
# More information here https://doc.rust-lang.org/cargo/guide/cargo-toml-vs-cargo-lock.html
Cargo.lock

# These are backup files generated by rustfmt
**/*.rs.bk

# MSVC Windows builds of rustc generate these, which store debugging information
*.pdb
```

---

## 构建产物

```gitignore
# 通用构建产物
build/
dist/
out/
target/
bin/
obj/
*.o
*.a
*.lib
*.dll
*.exe
*.app
*.dSYM/

# Web 构建产物
public/build/
static/js/
static/css/
static/media/

# 移动应用构建产物
*.apk
*.aab
*.ipa
*.app

# 桌面应用构建产物
*.dmg
*.pkg
*.msi
*.exe
*.deb
*.rpm
*.snap
*.appimage

# Docker 构建产物
*.tar
docker-compose.override.yml
```

---

## 日志文件

```gitignore
# 通用日志文件
*.log
logs/
log/
*.log.*
*.log-*

# 应用特定日志
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.pnpm-debug.log*

# 系统日志
syslog
*.log.gz
*.log.bz2
*.log.xz

# 错误报告
error.log
error_log
access.log
access_log
```

---

## 环境变量文件

```gitignore
# 环境变量文件
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
.env.staging
.env.*.local

# 配置文件
config.json
config.yaml
config.yml
secrets.json
secrets.yaml
secrets.yml

# API 密钥
.api-keys
api-keys.json
credentials.json
service-account.json
```

---

## 测试相关文件

```gitignore
# 测试覆盖率
coverage/
.nyc_output/
*.lcov
.coverage
htmlcov/
.tox/
.nox/
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

# 测试结果
test-results/
test-reports/
junit.xml
*.xml

# 端到端测试
/tests/e2e/videos/
/tests/e2e/screenshots/
cypress/videos/
cypress/screenshots/
playwright-report/
test-results/

# 性能测试
benchmark-results/
performance-reports/
```

---

## 缓存文件

```gitignore
# 通用缓存
.cache/
cache/
*.cache

# 构建工具缓存
.parcel-cache/
.vite/
.next/
.nuxt/
.gatsby/
.svelte-kit/

# 包管理器缓存
.npm/
.yarn/cache/
.pnpm-store/

# 编译器缓存
.tsbuildinfo
*.tsbuildinfo

# 运行时缓存
.eslintcache
.stylelintcache
```

---

## 临时文件

```gitignore
# 通用临时文件
*.tmp
*.temp
*.swp
*.swo
*~
.#*
\#*#

# 编辑器临时文件
*.orig
*.rej
*.bak
*.backup

# 系统临时文件
.Trash/
.Trashes
$RECYCLE.BIN/
```

---

## 数据库文件

```gitignore
# SQLite
*.sqlite
*.sqlite3
*.db
*.db3

# 其他数据库
*.mdb
*.accdb
*.dbf

# 数据库备份
*.sql
*.dump
*.backup
```

---

## 媒体文件

```gitignore
# 大型媒体文件（可选）
*.mp4
*.avi
*.mov
*.wmv
*.flv
*.webm
*.mkv

# 音频文件（可选）
*.mp3
*.wav
*.flac
*.aac
*.ogg
*.wma

# 图片文件（可选，根据项目需求）
# *.jpg
# *.jpeg
# *.png
# *.gif
# *.bmp
# *.tiff
# *.svg
# *.webp
```

---

## 压缩文件

```gitignore
# 压缩文件
*.zip
*.rar
*.7z
*.tar
*.tar.gz
*.tar.bz2
*.tar.xz
*.gz
*.bz2
*.xz
*.Z
*.lz
*.lzma
*.tlz
*.txz
```

---

## 完整配置模板

以下是一个包含所有常用忽略规则的完整 `.gitignore` 模板：

```gitignore
# =============================================================================
# 通用 .gitignore 配置模板
# 根据项目需求选择相应的部分
# =============================================================================

# 操作系统文件
# macOS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Icon?

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.lnk

# Linux
*~
.fuse_hidden*
.directory
.Trash-*

# IDE 和编辑器
# Visual Studio Code
.vscode/
*.code-workspace

# JetBrains IDEs
.idea/
*.iws
*.iml
*.ipr

# Sublime Text
*.sublime-project
*.sublime-workspace

# Vim
*.swp
*.swo
*~
.netrwhist

# AI 开发工具
# Cursor
.cursor/
*.cursor-workspace

# GitHub Copilot
.copilot/
.github-copilot/

# Kiro AI IDE
.kiro/
*.kiro-workspace
.kiro-specs/
kiro-config.json

# Windsurf
.windsurf/
*.windsurf-workspace

# Codeium
.codeium/
codeium.json

# 通义灵码
.tongyi/
.qwen/
tongyi-config.json

# AI 通用
.ai-cache/
.ai-sessions/
*.ai-chat
*.ai-session

# 依赖目录
# Node.js
node_modules/
npm-debug.log*
yarn-error.log*
.pnpm-debug.log*

# Python
__pycache__/
*.py[cod]
*.so
.Python
build/
dist/
*.egg-info/
.venv/
venv/

# Java
*.class
*.jar
target/
.gradle/

# PHP
vendor/
composer.lock

# Ruby
.bundle/
vendor/bundle/

# Go
vendor/
*.exe
*.dll
*.so
*.dylib

# Rust
target/
Cargo.lock

# 构建产物
build/
dist/
out/
public/build/
*.o
*.a
*.lib
*.exe
*.app
*.apk
*.ipa

# 日志文件
*.log
logs/
log/
error.log

# 环境变量和配置
.env
.env.local
.env.*.local
config.json
secrets.json
credentials.json

# 测试相关
coverage/
.nyc_output/
test-results/
cypress/videos/
cypress/screenshots/
playwright-report/

# 缓存文件
.cache/
.parcel-cache/
.vite/
.next/
.nuxt/
.eslintcache

# 临时文件
*.tmp
*.temp
*.swp
*.bak
.#*

# 数据库文件
*.sqlite
*.sqlite3
*.db

# 压缩文件
*.zip
*.rar
*.7z
*.tar.gz

# 大型媒体文件（可选）
# *.mp4
# *.mp3
# *.mov
```

---

## 使用建议

### 1. 项目初始化
在项目根目录创建 `.gitignore` 文件，并根据项目技术栈选择相应的配置。

### 2. 分层配置
对于大型项目，可以在不同目录创建特定的 `.gitignore` 文件：
- 根目录：通用配置
- 前端目录：前端特定配置
- 后端目录：后端特定配置

### 3. 团队协作
确保团队成员使用相同的 `.gitignore` 配置，避免不必要的文件被提交。

### 4. 定期更新
随着项目发展和新工具的使用，定期更新 `.gitignore` 配置。

### 5. 全局配置
可以设置全局 `.gitignore` 文件：
```bash
git config --global core.excludesfile ~/.gitignore_global
```

---

## 相关资源

- [GitHub .gitignore 模板](https://github.com/github/gitignore)
- [gitignore.io](https://www.toptal.com/developers/gitignore) - 在线生成工具
- [Git 官方文档](https://git-scm.com/docs/gitignore)

---

**文档版本**: v1.0  
**最后更新**: 2025年1月  
**维护者**: AI 助手  

> 💡 **提示**: 根据项目实际需求选择合适的配置，避免过度忽略重要文件。