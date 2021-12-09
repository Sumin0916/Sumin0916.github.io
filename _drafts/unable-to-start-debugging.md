---
title:  "unable to start debugging. unexpected gdb output from command -environment-cd 해결 방법" 

categories:
  - PS
tags:
  - [PS, tree, programmers]

toc: true
toc_sticky: true

date: 2021-12-07
last_modified_at: 2021-12-07
---

처음으로 VSCode를 사용하여 C/C++를 코딩하기 위해 개발 환경을 세팅 완료하고 실행하였을 때 발생한 오류이다. 며칠 동안 Stackoverflow, github 등 여러 해외 사이트들을 찾아다니며 해결법을 알려고 했지만 결과는 소용이 없었다.

 
해외 사이트들에 있는 답변은 장황한 코드를 붙여 넣거나 여러 가지의 설정을 일일이 변경하기에 아닌 것 같다고 판단하였다. 같은 오류명이지만 처해있는 상황이 너무 달라 명확한 해결 방법을 찾을 수 없었던 것 같다. 그리하여 나와 같은 상황에 처해있는 사람들을 위해 글을 남기게 되었다.

(쓴이는 첫 설치를 끝내고 첫 코드 실행을 앞둔 상황이었다.)

 
참고로 OS는 win 10을 사용하였고 컴파일러는 minGW를 설치하여 g++를 사용하였다.

 
 해결 방법은 간단하였다

코드가 있는 파일 경로에 한글이 있어서 문제가 발생한 것이다. (내 사라진 3시간...)

경로 내에 한글로 된 파일이나 폴더가 있으면 컴파일러가 경로를 인식하지 못하는 것 같다.

ex)  C:\Users\Username\Desktop\작업공간\test.c -> C:\Users\Username\Desktop\workspace\test.c 이런 식으로 수정을 해주면 된다.