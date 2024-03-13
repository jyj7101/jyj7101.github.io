---
layout: post
title: Vertex Smear Effect
katex: True
categories: [Graphic, Programming, Shader]
---
# 머리

버텍스 스미어. 

텔레포트나 고속으로 이동하면 나타나는 그 잔상같은 효과를 말하는 것입니다.

gist에 있는 것들 좀 옮겨옵니다. 작성일 2023 10 23

# 몸

## 코드로!

[https://gist.github.com/ashuatz/323f2ea31d77982983fd890fc3b263b9](https://gist.github.com/ashuatz/323f2ea31d77982983fd890fc3b263b9)

![277265060-c284a77e-fb77-46d5-8198-b8c9ad1e701d.gif](/assets/VertexSmear/277265060-c284a77e-fb77-46d5-8198-b8c9ad1e701d.gif)

코드로 제어하는 버텍스 스미어 이펙트!

Job을 통해 병렬로 메시의 버텍스를 조절합니다.

영향을 받을 콜라이더를 통해 몇몇위치를 둘 수 있습니다.

자세한건 gist 참조…!

## 쉐이더로!

[https://gist.github.com/ashuatz/323f2ea31d77982983fd890fc3b263b9](https://gist.github.com/ashuatz/323f2ea31d77982983fd890fc3b263b9)

![280167006-f01c33da-2496-4528-8c58-5a71b56cd8e1.gif](/assets/VertexSmear/280167006-f01c33da-2496-4528-8c58-5a71b56cd8e1.gif)

익숙한 그것! 

쉐이더는 일반적인 구현이니 

![Untitled](/assets/VertexSmear/Untitled.png)

자세한건! gist! 참조!

# 꼬리

포스트 파편화가 너무많이 되어있어서 (노션, 블로그, gist, 카톡(주로 남의 개인톡에 자랑함) 등)

블로그에 한번 합칩니다.. 이래도 누락되는게 많고 못쓰는 것도 많아 슬프군요.