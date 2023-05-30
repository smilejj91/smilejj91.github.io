---
title:  "How to create plymouth theme"

categories:
  - OS
tags:
  - plymouth
  - splash
  - debian
  - os
# 목차
toc: true
toc_sticky: true
---

# How to create plymouth theme

## 1. splash screen

* 프로그램의 시작 또는 종료 구간에 보여주는 로딩 화면이며, OS에서는 부팅/종료/다시 시작 등 system의 시작과 종료 구간에서 사용

## 2. plymouth

* debian에서는 splash screen을 보여주기 위해 plymouth 라는 패키지를 이용하고 있음

## 3. plymouth theme

* debian에서는 기본적인 theme을 가지고 있는 desktop-base 패키지에서 제공(grub, login, wallpaper, plymouth)

## 4. file tree

```bash
usr
└ share
  └ plymouth
    └ themes
      └ {Theme Name}
        ├ {Theme Name}/{Theme Name}.plymouth
        └ {Theme Name}/{Theme Name}.script
```

## 5. create custom plymouth theme

### Step 1. create .plymouth file 

> ImageDir : .script에서 image를 loading할 root directory path

```bash
[Plymouth Theme]
Name=SMILEJJ
Description=SMILEJJ Splash
ModuleName=script

[script]
ImageDir=/usr/share/plymouth/themes/smilejj
ScriptFile=/usr/share/plymouth/themes/smilejj/smilejj.script
```

### Step 2. create .script file

```bash
...
Plymouth.SetDisplayPasswordFunction(display_password_callback); // 암호 입력 화면 (plymouth ask-for-password handler)
Plymouth.SetRefreshFunction (refreshHandler); // 1초에 50번 불리는 handler
Plymouth.SetUpdateStatusFunction (statusHandler); // system 상태 출력 화면 (plymouth update handler)
Plymouth.SetMessageFunction (message_callback); // 메시지 출력 화면 (plymouth display-message handler)
Plymouth.SetSystemUpdateFunction(progress_callback); // 오프라인 업데이트 화면 (plymouth system-update handler)
```

### Step 3. apply custom plymouth theme 

```bash
$ sudo plymouth-set-default-theme -R smilejj
```

## 6. link

[Plymouth Document](https://www.freedesktop.org/wiki/Software/Plymouth/Scripts/)
