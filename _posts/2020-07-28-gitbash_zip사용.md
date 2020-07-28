---
title: Git bash에서 zip 명령어 사용하기
layout: post
date: '2020-07-28 21:18:00'
author: 진혀크
tags: Gitbash zip linux
cover: "/assets/instacode.png"
categories: linux
---

Gitbash에서 Linux를 공부하다가 만난 문제이다.

    zip: command not found

위의 에러가 뜨며 zip 명령어가 실행되지 않았다. 사실 tar로 압축할까 하다가 zip이 쓰고 싶어 방법을 찾아봤다.

방법은 생각보다 어렵지 않다.

<https://sourceforge.net/projects/gnuwin32/files/>에 접속해서 zip과 bzip2에 들어간다.

<img src="{{ site.baseurl }}/assets/use_zip1.PNG" width = "70%" height ="50%" title="webpage_capture" class="picture">

거기서 zip-3.0-bin.zip 과 bzip2-1.0.5-bin.zip 를 다운 받는다. (작성 기준 가장 최신 버전인데 가장 최신 버전에서 다운 받으면 될 것 같다.)

압축을 푼 뒤 zip.exe파일과 bzip2.dll 파일을 찾는다. bin폴더에 들어있을 것이다.

2개의 파일을 git/usr/bin 에 넣어주면 끝! (git/bin이 아니고 git/usr/bin이다.)
<img src="{{ site.baseurl }}/assets/git_route.PNG" width = "70%" height ="50%" title="webpage_capture" class="picture">

터미널을 재부팅한 후 터미널에서 zip 명령어를 실행해보면 잘 되는 것을 확인할 수 있을 것이다!
