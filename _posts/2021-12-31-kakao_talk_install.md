---
title     : "Ubuntu 18.04에 Kakao Talk 설치"
tags        : [ubuntu]
style       : fill # fill / border (choose one only)
color       : info # primary / secondary / success / danger / warning / info / light / dark (choose one only)
description : # Write post description here, or it will be the first 25 words of the post's body.
---

Ubuntu 18.04에 카카오톡 설치 방법 입니다.

## Install Wine

wine은 ubuntu에서 window프로그램(.exe)을 실행시켜주는 호환성 레이어(가상머신은 아니지만 윈도우 명령어를 우분투로 매핑시켜주는)이다.

- reference: https://wiki.winehq.org/Ubuntu

- ```bash
    sudo dpkg --add-architecture i386
    wget -nc https://dl.winehq.org/wine-builds/winehq.key
    sudo apt-key add winehq.key
    sudo add-apt-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main'
    sudo apt update
    sudo apt install --install-recommends winehq-stable
    ```

## Install Winetricks

font 관련 문제발생을 막기 위해 winetricks을 사용할 수 있다.

- ```bash
  sudo apt install winetricks
  winetricks corefonts
  ```

## Install KakaoTalk

카카오톡 홈페이지에서 윈도우 설치파일을 받고 영어버전으로 설치한다. 
(한글버전으로 설치시 wine이 꼬여서 카카오톡 로그인 에러가 발생한다. 에러가 발생시, wine을 깔끔하게 삭제후 재설치해야함.)

- ```bash
    # 다운로드:
    wget https://app-pc.kakaocdn.net/talk/win32/KakaoTalk_Setup.exe
    # 설치:
    wine KakaoTalk_Setup.exe
    ```

## Execute KakaoTalk

- ```bash
    cd ~/.wine/drive_c/Program\ Files\ \(x86\)/Kakao/KakaoTalk/
    LC_ALL=ko_KR.utf8 wine KakaoTalk.exe
    ```

## Put Wine's System Tray in Ubuntu's System Tray

- ```bash
    sudo apt-get install gnome-tweaks gnome-shell-extensions
    sudo apt install -f gnome-shell-extension-top-icons-plus
    ```

- (아마 재부팅 필요) Tweaks > Extentions > Topicons Plus On

## How to Reset Wine?

설치중에 wine이 꼬였다면, 설정 파일 모두 삭제후 재설치해야한다.

- ```bash
    rm -Rf $HOME/.wine*
    rm -f  $HOME/.local/share/mime/packages/x-wine-e*
    rm -f  $HOME/.local/share/mime/application/x-wine-*
    rm -f  $HOME/.local/share/icons/hicolor/*/apps/application-x-wine-*
    rm -f  $HOME/.local/share/applications/wine-*
    rm -f  $HOME/.local/share/desktop-directories/wine-*
    rm -f  $HOME/.local/share/icons/hicolor/*/apps/7765_*
    rm -f  $HOME/.config/menus/applications-merged/wine-*
    sudo apt purge wine*
    ```
