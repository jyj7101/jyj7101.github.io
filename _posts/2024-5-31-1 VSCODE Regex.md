---
layout: post
title: VSCODE 정규표현식 Replace
katex: True
categories: [Programming,Regex]
---

방금 글쓰다가 짜증이 나서 생각난김에 씁니다.

regex는 기억하다가도 안쓰면 까먹는단 말이죠...

vscode(및 vs)에서는 정규표현식을 사용해서 replace를 할 수 있는데,

괄호나 대괄호, 따옴표등 단어를 랩핑하는 경우 굉장히 귀찮습니다.

# 머리

\b로 단어경계를 지정하고, \w로 문자를 지정하며, +를 사용하여 다음 \b가 나올때까지 매칭합니다.

그다음 $0을 사용해서 매칭된 단어를 삽입할 수 있습니다.

![Untitled](/assets/vscode_regex/Untitled.png)

이거하고 `도 붙어있을테니까 알아서 잘 떼주면... 

![Untitled](/assets/vscode_regex/Untitled%201.png)

편-안

# 추가

$는 각 캡처 그룹에 해당하는 매칭된 문자열을 의미하므로,

$1 $2 … 이게 매칭 그룹의 순번에 에 해당됩니다. 

예시론 다음과 같겠네요

![regex.gif](/assets/vscode_regex/regex.gif)

# 참조

[https://regex101.com/](https://regex101.com/)

여기서 정규표현식을 테스트해보세요