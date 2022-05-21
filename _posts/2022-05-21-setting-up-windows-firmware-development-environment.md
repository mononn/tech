---
layout: post
title: 윈도우즈 펌웨어 개발환경 설정
subtitle: 윈도우즈에서 ARM Cortex-M 기반 펌웨어를 빌드하기 위한 환경 설정 방법을 안내합니다.
date: 2022-05-21 23:50:00 +0900
---

윈도우즈 머신은 거의 사용하지 않지만, 동료 또는 클라이언트의 요청이 있는 경우
윈도우즈에서 펌웨어를 빌드할 수 있도록 환경을 셋팅할 필요가 있었습니다. 간단한
가이드를 작성합니다.

## 설치
1. [vscode 설치](https://code.visualstudio.com/)
    * [c/c++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
    * [windows-arm-none-eabi extension](https://marketplace.visualstudio.com/items?itemName=metalcode-eu.windows-arm-none-eabi)
        * 버전이 낮기 때문에 [ARM 공식 사이트](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/downloads)에서 최신 버전 다운로드 받아 사용하도록 가이드하기
            * 설치 완료시 `Add path to environment variable` 반드시 체크하기
2. [msys2 설치](https://www.msys2.org/)
    * `pacman -Syu`
    * `pacman -S make`
3. 터미널 환경 설정
    * [vscode 터미널 설정](https://stackoverflow.com/questions/45836650/how-do-i-integrate-msys2-shell-into-visual-studio-code-on-window) 하거나
    * 시스템 환경 변수 설정
        * `c:\msys64\mingw64\bin`
        * `c:\msys64\usr\bin`
    * 코드베이스에 vscode 터미널 설정을 포함해 이 스텝은 스킵할 수 있도록 할 것

## 기타
* docker 를 사용하는 게 설치는 가장 쉽지만, 빌드 시간이 너무 오래 걸림
* 디버깅과 플래시 설정 추가하기
    * [https://code.visualstudio.com/docs/cpp/config-mingw](https://code.visualstudio.com/docs/cpp/config-mingw)
* 디버깅 플러그인
    * Cortex-Debug
    * [https://wiki.segger.com/J-Link_Visual_Studio_Code](https://wiki.segger.com/J-Link_Visual_Studio_Code)
