# 5. Depth(a.k.a  Z-Buffer)

---

# 머리말

---

하하 하필 최근 뎁스문제를 좀 겪고 있었는데 강의조차 뎁스가 나오는군요!

운명이라 생각하고 받아들였습니다.

# **3차원 NDC공간**

---

## NDC 공간의 확장?

---

안타깝게도 2차원공간으로만 사용하게 된다면 앞과 뒤의 개념이 없어지게됩니다.

그렇게되면 뒤에있는 오브젝트가 그려진다던지, 

반만그려진다던지 하는 여러가지 문제가 발생합니다.

이걸 보통 뎁스(Depth)문제라고 하는데, 쉽게보자면 아래와 같이 개판이 되는 겁니다.

자, 익숙한 유니티 샘플신에서의 그냥 일방적 렌더링을 한번 봅시다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled.png)

개판이 나버린 뎁스(Depth)

땅 밑에있는 나무골자가 드러나고, 내부 파이프가 앞에그려지고, 심지어 파란색 무언가는 저 선반에 가려 보이지도 않습니다(다시 그려졌을지도 모르죠)

이번에 설명하고자 하는것은 이것입니다.

일단 결과론적으로

여기서 이제 화면에 어떻게 그릴 것인지 깊이를 설정하게되면,

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%201.png)

유니티에서의 화면좌표값 (ScreenPosition.w)

이렇게 (지금은 시각화된) 깊이정보를 그리게 되고, 최종적으로 Comparision 하여 그리게 됩니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%202.png)

뎁스테스트가 된 결과물

짜잔

이번에 진행할 내용은 전체적으로 위 내용에 대한 것입니다.

하나하나 차근차근 진행해 볼까요?

## 상용 엔진의 각 공간에 대한 좌표계

---

그래픽 API

---

**DirectX**

- 로컬 : 왼손 좌표계
- 월드 : 왼손 좌표계
- 뷰 : 왼손 좌표계
- NDC : 왼손 좌표계

**OpenGL**

- 로컬 : 오른손좌표계
- 월드 : 오른손 좌표계
- 뷰 : 오른손 좌표계
- NDC : 왼손 좌표계

[게임 엔진](https://www.notion.so/1-3D-Transform-6f2c493c074349c4bbda0c43e48b08ed)

---

**[Unreal4 Engine](https://docs.unrealengine.com/4.26/en-US/Basics/Actors/CoordinateSpace/)**

- 로컬 : 왼손
- 월드 : 왼손
- 뷰 : 왼손

- NDC,화면 : 왼손 (X[-1,1] Y[-1,1] Z[0,1], 단 RHI 사용시 -1,1로 변환 필요)

**Unity3D Engine**

- 로컬 : 왼손
- 월드 : 왼손
- 뷰 : 왼손

- [클립](https://docs.unity3d.com/Manual/SL-PlatformDifferences.html#:~:text=Direct3D-like%3A%20The%20clip%20space%20depth%20goes%20from%20%2B1.0%20at%20the%20near%20plane%20to%200.0%20at%20the%20far%20plane.%20This%20applies%20to%20Direct3D%2C%20Metal%20and%20consoles)
    
    > **Clip space coordinates**
    > 
    > 
    > Similar to Texture coordinates, the clip space coordinates (also known as post-projection space coordinates) differ between Direct3D-like and OpenGL-like platforms:
    > 
    > - **Direct3D-like**: The clip space depth goes from +1.0 at the near plane to 0.0 at the far plane. This applies to Direct3D, Metal and consoles.
    > - **OpenGL-like**: The clip space depth goes from –1.0 at the near plane to +1.0 at the far plane. This applies to OpenGL and OpenGL ES.
    - Direct3D Z[1,0]
    - OpenGL Z[-1,1]

## NDC 공간의 Z범위가 [-1,1]일때의 투영 행렬

---

우선 투영행렬에 뷰행렬 좌표를 곱셈 연산을 해줍니다.

$$
P\cdot\ v_{view}=\left[\begin{matrix}\frac{d}{a}&0&0&0\\0&d&0&0\\0&0&k&l\\0&0&-1&0\\\end{matrix}\right]\left[\begin{matrix}v_x\\v_y\\v_z\\1\\\end{matrix}\right]=\left[\begin{matrix}\frac{d}{a}\cdot v_x\\d\cdot v_y\\?\\-v_z\\\end{matrix}\right]$$

여기서  $(P\cdot\ v_{view})_z$ 는 $(P\cdot\ v_{view})_w$로 나누었을 때 near : -1, far : 1이 나와야하기 때문에 

$$
(0,0,-n,n)\quad(0,0,f,f)$$

와 같습니다.

위를 얻기 위해 미지수 $k,l$을 아래의 식에서 연립방정식을 얻어냅니다.

$$P=\left[\begin{matrix}\frac{d}{a}&0&0&0\\0&d&0&0\\0&0&k&l\\0&0&-1&0\\\end{matrix}\right]\left[\begin{matrix}0\\0\\-n\\1\\\end{matrix}\right]=\left[\begin{matrix}0\\0\\-n\\n\\\end{matrix}\right]$$

$$P=\left[\begin{matrix}\frac{d}{a}&0&0&0\\0&d&0&0\\0&0&k&l\\0&0&-1&0\\\end{matrix}\right]\left[\begin{matrix}0\\0\\-f\\1\\\end{matrix}\right]=\left[\begin{matrix}0\\0\\f\\f\\\end{matrix}\right]$$

$$-kn+l=-n{\quad}-kf+l=f\\
k=\frac{(n+f)}{(n-f)}{\quad}l=\frac{2nf}{(n-f)}$$

(이걸 해냅니다)

연립 방정식을 풀어내면 아래와 같이 값을 구할 수 있습니다.

$$\therefore P=\left[\begin{matrix}\frac{d}{a}&0&0&0\\0&d&0&0\\0&0&\frac{n+f}{n-f}&\frac{2nf}{n-f}\\0&0&-1&0\\\end{matrix}\right]$$

## NDC 공간의 Z범위가 [0,1]일때의 투영 행렬

---

프로그래머특징 : 공통부분은 그냥 날로먹고 사용합니다.

중간까지는 똑같으니 무시하고 연립방정식만 바꿔볼까요?

$$-kn+l=0{\quad}-kf+l=f\\
k=\frac{f}{(n-f)}{\quad}l=\frac{nf}{(n-f)}$$

$$\therefore P=\left[\begin{matrix}\frac{d}{a}&0&0&0\\0&d&0&0\\0&0&\frac{f}{n-f}&\frac{nf}{n-f}\\0&0&-1&0\\\end{matrix}\right]$$

# **원근투영보정**

---

아 그러면 이제 렌더링이 모두 끝났냐고요? 아니요... 그렇게 만만하지 않습니다.

수학은 언제나 저희에게 난관과 시련을 선사하죠.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%203.png)

딱봐도 알겠죠? 무언가 잘못되었습니다.

어떻게 해결하냐고요? 키워드는 제목입니다.

## 원근투영보정 간단 정리

---

이유는 간단합니다. 

선형적이지 않은데 선형적으로 처리했기 때문이죠.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%204.png)

<화면상에선 선형이라고 하더라도, 실제론 선형적이지 않은 케이스>

상대가 약을 판다면 저희도 약을 팔아야합니다. 

간단합니다. 

뷰 좌표계에서 각 점의 x, y는 z로 나뉘어지게 되는데, 

그 비율에 맞추어 처리한다고 생각하면 좋습니다. 

대신 무게중심좌표계는 1이라는 값이 나와야 UV에 맵핑을 하기 때문에, 

변환된 가중치들을 더한 값이 높은 확률로 1이 아니게 되므로,

해당 비율로 나누게 된다면 최종적으로 다시 '비율적으로 맞는 무게중심좌표계' 가 나오는 것입니다.

### 원근투영보정의 수식 유도

---

일단 [무게중심좌표](https://www.notion.so/11-Convex-Line-Clipping-Mesh-ebcc20c2d47f493aa66a637b654d8cdf)를 복기해봅시다.

버텍스 $v_0,v_1,v_2$ 에 대해 각 무게중심좌표를 가중치라는 의미로 $weight, w$로 표기하여  $w_0(s),w_1(t),w_2(1-s-t)$ 라고 정의합니다.

그 후, 

> 뷰 좌표계에서 각 점의 x, y는 z로 나뉘어지게 되는데,
> 
> 
> 그 비율에 맞추어 처리한다고 생각하면 좋습니다. 
> 

라고 한 방식대로 일단 각 정점의 거리값으로 나눠줍니다.

$$ w_0,w_1,w_2\mapsto 
w_0 \cdot \frac{1}{{v_0}_w},
w_1 \cdot \frac{1}{{v_1}_w},
w_2 \cdot \frac{1}{{v_2}_w}$$

> 대신 무게중심좌표계는 1이라는 값이 나와야 UV에 맵핑을 하기 때문에,
> 
> 
> 변환된 가중치들을 더한 값이 높은 확률로 1이 아니게 되므로,
> 
> 해당 비율로 나누게 된다면...
> 

라고 말한 해당비율로 나누기 위한 값. 즉 나눠진 가중치의 합을 구해볼까요?

$$S = \sum_{i=0}^{3} w_i\cdot \frac{1}{v_i}_w$$

```cpp
//수학 잘 모르니 코드로 표현해봅시다.
sum = 0;
for (int i = 0; i < 3; ++i)
{
   sum += w[i] * (1 / v[i].w);
}
```

그렇게 구해진 $S$로 나눔으로써($S$의 역원을 곱함으로써) 끝을 내게 합니다.

$$ w_0,w_1,w_2\quad\mapsto \quad
w_0 \cdot \frac{1}{{v_0}_w \cdot S},\quad
w_1 \cdot \frac{1}{{v_1}_w\cdot S},\quad
w_2 \cdot \frac{1}{{v_2}_w\cdot S}$$

..원래 수학은 수식으로 보는 것보다 글로 읽는게 쉽습니다.

다시 짧게 정리하면

1. 비율 맞춰줘야하니까 거리로 나누고
2. 원본은 합이 1이여야 하니까 안 맞는 만큼 나눠주고

...결국 리매핑입니다. 어때요 참 쉽죠?

# **깊이값(Depth)의 활용**

---

## 깊이 버퍼(Depth Buffer)와 활용

---

하하

깊이 테스트란 무엇인가! 

간단하게는 거리값을 각 픽셀마다 테스트하여, 지금 그리고있는 메시의 해당 화면좌표에 저장된 값과 비교하여 갱신하거나, 패스하는 방법입니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%205.png)

유니티 깊이 텍스처

> [**깊이버퍼(Depth)]()**
> 
> 
> ![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%201.png)
> 
> 유니티에서의 화면좌표값 (ScreenPosition.w)
> 
> 이렇게 (지금은 시각화된) 깊이정보를 그리게 되고, 
> 최종적으로 Comparision 하여 그리게 됩니다.
> 

이제 이부분을 조금 자세하게 설명하려 합니다.

렌더 루프에서 결국 메쉬→삼각형→삼각형내부의점→픽셀색상→화면위치에 그리기

에서 **화면위치에 이미 그려진 점이 있다면?** 이라는 개념으로 접근합니다.

초기화 : 화면크기만큼의 버퍼를 생성

1. 렌더루프 : 현재 그리고 있는 화면의 픽셀의 저장된 거리값과 비교
    1. 만약 비교값이 참이라면 
        1. 버퍼에 거리값을 저장
        2. 그리기
    2. 만약 비교값이 거짓이라면
        1. 그리지 않고 넘기기(Continue;)

여기서 왜 설명을 '비교값이 ~라면' 의 꼴로 두었는지를 봐야합니다.

이 비교(comparision) 라는 것은 변경될 수 있습니다.

- 유니티의 경우 [ZTest](https://docs.unity3d.com/kr/530/Manual/SL-CullAndDepth.html#:~:text=%EB%82%B4%EC%9A%A9%EC%9D%80%20%EB%8B%A4%EC%9D%8C%EC%9D%84%20%EC%B0%B8%EC%A1%B0%ED%95%98%EC%8B%AD%EC%8B%9C%EC%98%A4.-,ZTest,-ZTest%20Less%20%7C%20Greater) 값을,
- 다이렉트X의 경우 [D3D11_COMPARISON_FUNC](https://docs.microsoft.com/en-us/windows/win32/api/d3d11/ne-d3d11-d3d11_comparison_func)을
- OpenGL의 경우 [glDepthFunc](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glDepthFunc.xhtml) 를

사용하여 변경할 수 있습니다.

일반적인경우

- Less
    - CK - 소프트렌더러(2021)
    - 다이렉트X
    - OpenGL
- LessEqual
    - 유니티
    

을 사용합니다.

즉 더 가까운것만 그리는 것이죠.

그리고 이런 뎁스를 저장해둔 정보를 텍스처로써 저장해 사용할 수 있습니다.

### 활용

---

그리고 이를 구워둘 경우 추후 활용할 수 있죠.

요즘 하는 게임들에서 어 이부분은 이렇게 구현하면 되지않을까? 하는 부분들을 찾아서 구현하는게 취미라... 그 중 뎁스를 사용한 것들을 일부 소개할려합니다.

첫번째로, Scene Depth에서 screenPosition.w의 일부를 제거한 값을 빼, 접점영역을 구할 수 있습니다. 

3D게임에서 영역을 표시하거나, 

바다의 거품, 물의 끝쪽의 깊이색상을 낮추는 곳에 사용할 수 있습니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%206.png)

뎁스를 사용한 물체의 접(intersection) 표시 쉐이더

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%207.png)

뎁스를 사용한 물 쉐이더

두번째로, 아까 소개한 Depth비교를 변경하여, 가려진 오브젝트를 다른 머테리얼로 렌더링할 수 있죠.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%208.png)

깊이테스트를 Less(가까움!)이 아닌 Greater(멀어!) 일 경우,

다른 머테리얼을 사용하서 그려줍니다. 저는 실험적으로 만들어둔 디더 머테리얼을 넣었습니다.

아래의 큐브를 봐주세요 (그냥 unlit 넣을걸 그랬나봅니다)

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%209.png)

디더특징 : 축소하면 안보임

그러면 이렇게 가려진부분만 한번 더 그려서, 다른 방식으로 보여질 수 있게 나오게 됩니다.

첫번째의 경우 거리에 초점을 맞춘 케이스고,

두번째의 경우 그려진다, 그려지지않는다 라는 비교에 초점을 맞춘 케이스입니다.

이렇게 다양한 방법으로 깊이를 사용할 수 있는데.....

....

## [Z-Fighting](https://en.wikipedia.org/wiki/Z-fighting)(Stitching)

---

하지만 문제가 발생합니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2010.png)

두개의 렌더대상이 카메라와 아주 유사한 거리를 가질때 발생하는, **어떤 것이 더 멀리 떨어져있는지 정확히 구분할 수 없을 때** 무작위로 그려지는 현상을 말합니다.

보통 부동소숫점 정밀도와 고정소숫점 반올림 오류로 발생하죠.

현실에서도 간혹 발생하는 문제죠..

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2011.png)

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2012.png)

어떻게 해결하냐고요?

[https://twitter.com/freyaholmer/status/799602767081848832](https://twitter.com/freyaholmer/status/799602767081848832)

농담이자 진담입니다.

## 깊이 버퍼의 선형화(Linearize Depth)

---

깊이버퍼를 선형화 하지 않으면 생기는 문제부터 말씀을 드리는게 인지상정.

선형화 하지 않을 경우, 아무래도 정밀도를 손해보기때문에 

부드러운 처리가 되기 힘든 경향이 있습니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2013.png)

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2014.png)

Z-fighting 의 경우도 마찬가지입니다. 

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2015.png)

선형과 Raw값을 비교해보면 쉽게 알 수 있죠.

그렇기때문에 저희는 선형화를 해야합니다.

### 1. 처음부터 넣을 때 선형값을 사용한다.

가장 편한방법은 처음부터 선형값을 사용하는 것입니다.

실제 무게중심좌표를 구하는 타이밍에서 사용하였던 그 거리값을 가져와서 사용하는 방법입니다.

$$z^{-1} - near : far - near
\\depth = \frac{(z^{-1} - near)}{(far - near)}\\\quad\\$$

유니티에서 ScreenPosition.w를 받는 느낌이죠.

### 2. 어려운길을 걷는다.

이걸 이제 나온값을 되돌릴려고한다면... $[-1,1]$의 영역에 대해서,

아까 봤던 [매트릭스]()를 참조하여, 

$$z_n=z_{normalized} = 2 \cdot dis - 1\\
\quad
\\
depth_{linear} = \frac{2 \cdot near \cdot far}{far + near - z_n \cdot(far - near)}$$

를 처리해주면 됩니다.

![Untitled](5%20Depth(a%20k%20a%20Z-Buffer)%20c9a16f9c14974a7ea3b0500bfee922b2/Untitled%2016.png)

Rmx

[[참조]](http://web.archive.org/web/20130416194336/http://olivers.posterous.com/linear-depth-in-glsl-for-real)