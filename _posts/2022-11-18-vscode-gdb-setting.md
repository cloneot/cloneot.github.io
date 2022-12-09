---
title: "vscode 빌드, 디버깅 세팅"
layout: post
---

## vscode 빌드, 디버깅 세팅

https://code.visualstudio.com/docs/cpp/config-msvc



-environment-cd 에러, invalid argument: 한글 문제 (win11 이슈)

https://ttum.tistory.com/4



https://velog.io/@embeddedjune/VSCode-WSL2%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-CC-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%B9%8C%EB%93%9CGDB-%EB%94%94%EB%B2%84%EA%B9%85-%EB%B0%A9%EB%B2%95-%EB%A9%94%EB%AA%A8



breakpoint를 걸면 에러가 나는 경우 gdb 버전 문제일 수도 있음

https://github.com/microsoft/vscode-cpptools/issues/10018



변수 디폴트는 hex라서 proposed extension API라는 세팅을 추가로 해줘야 편하게 볼 수 있음

https://code.visualstudio.com/updates/v1_49#_contributable-context-menu-for-variables-view





## C/C++ extension을 깔지 못하는 경우

tasks.json 파일에 task를 직접 추가



command: "/usr/bin/g++ \${file} -o \${fileBasenameNoExtension} & \${fileDirname}/${fileBasenameNoExtension} < \${fileDirname}/input.txt > \${fileDirname}/output.txt"
group: { "kind": "build", "isDefault": true }

같은 것만 수정해주면 됐던 것 같다. 



## C++20

contains 등을 사용하기 위해서는 -std=c++20 옵션이 필요하다. 

std 디폴트 옵션은 g++ 버전에 따라 고정되어 있는듯 하다. code runner setting에서 -std=c++20 옵션을 추가하는 게 가장 깔끔한 방법 같다. 

