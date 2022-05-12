---
layout: post
title: 6. Matrix - Scale, Shift, Rotate
katex: True
categories: [그래픽 프로그래밍 2D,assignment]
---

# 6. Matrix : Scale, Shift, Rotate

---

### 표준기저벡터의 변환 과정 개념에서 다음의 행렬을 유도하시오.

설명에 앞서 공간 정의부터 진행하겠습니다.

데카르트 좌표계 (Cartesian coordinate system)를 기본으로,

두 기저벡터($$\vec{e_1},\vec{e_2}$$) 에 대해 기본값은 다음과 같다고 가정합니다.

($$\vec{e_1},\vec{e_2}$$) 는 각각 ($$\vec{right},$$ $$\vec{up}$$)이라고 생각하면 될 것 같습니다.

$$
\vec e_1 = \begin{bmatrix}
1 \\
0
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
0 \\
1
\end{bmatrix}
$$

### 크기 행렬

x축으로 $$a_x$$ 배 만큼, y축으로 $$b_y$$배 만큼 공간을 늘린다면,

이는 해당축에 맞는 기저벡터에 스칼라배를 한것과 같다고 생각합니다.

기저벡터의 성분에 스칼라배를 통한 선형조합을 만드는 것을 반대로 말한게 되네요

따라서 두 기저벡터는 다음과 같습니다.

$$
\vec e_1 = \begin{bmatrix}
1 \cdot a_x \\
0
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
0 \\
1 \cdot b_y
\end{bmatrix}
$$

이렇게 각 축에 해당되는 기저벡터에 스칼라배를 통하여 공간을 확장할 수 있습니다.

이는 행렬에 넣게되면 이런 형태가 되는데,

$$

\bold A\vec v =\begin{bmatrix}
a_x  & 0\\
0  & b_y
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
a_xx\\
b_yy
\end{bmatrix} = \bold u
$$

사실 묶어놓고 본다면 아래와 동일합니다. 
($$\vec{e_1}$$ 과 그 대괄호는 현재 이미지편집을 할 수 없어, 분류를 위해 표기하였습니다. 
 위 수식에 추가로 그려놓은 주석이라고 이해해주시면 감사하겠습니다.)

$$

\bold A\vec v =\begin{bmatrix}
\begin{bmatrix}^{\vec{e_1}}\\
a_x \\
0
\end{bmatrix} & 
\begin{bmatrix}^{\vec{e_2}}\\
0 \\
b_y
\end{bmatrix}
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
a_xx\\
b_yy
\end{bmatrix} = \bold u
$$

따라서 기본 행렬은 다음과 같습니다.

x축의 크기인 $$x_{scale}$$과 y축의 크기인 $$y_{scale}$$로 표기하였습니다.

$$
\begin{bmatrix}
x_{scale} & 0\\
0 & y_{scale}
\end{bmatrix}
$$

### y축 밀기 행렬

y축 밀기 행렬. 즉 $$\vec{e_1}$$의 y성분이 0이 아니라 특정 수 $$a_y$$가 된다면, 

y축으로 $$a_y$$만큼 밀린 공간이 나타날 것 입니다.

따라서 두 기저벡터는 다음과 같이 나타낼 수 있습니다

$$
\vec e_1 = \begin{bmatrix}
1  \\
a_y
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
0 \\
1 
\end{bmatrix}
$$

그것을 다시 크기행렬과 같이 두 기저벡터가 들어있는 행렬로써 표현하자면,

$$

\bold A\vec v =\begin{bmatrix}
1  & 0\\
a_y  & 1
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
x\\
a_y\cdot x + y
\end{bmatrix} = \bold u
$$

과 같이 표현할 수 있습니다.

따라서 y축 밀기 행렬은 

$$
\begin{bmatrix}
1 & 0\\
y_{shift} & 1
\end{bmatrix}
$$

과 같이 이해할 수 있다고 생각합니다.

### 시계 방향 90도 회전 행렬, 반시계 방향 90도 회전 행렬

시계방향 90$$^{\circ}$$ 회전이라. 즉 두 기저벡터의 위치관계는 바뀌지 않으며, 방향이 바뀐다 볼 수 있습니다. 따라서 다음과 같은 두 기저벡터를 정의할 수 있겠습니다

$$
\vec e_1 = \begin{bmatrix}
0  \\
-1
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
1 \\
0 
\end{bmatrix}
$$

다시 이걸 두 기저벡터가 들어있는 행렬로써 표현하면,

$$

\bold A\vec v =\begin{bmatrix}
0  & 1\\
-1  & 0
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
y\\
-x
\end{bmatrix} = \bold u
$$

로 볼 수 있습니다.

그렇다면 반시계방향90$$^{\circ}$$는 어떨까요?

두 기저벡터를 정의한다면 다음과 같습니다.

$$
\vec e_1 = \begin{bmatrix}
0  \\
1
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
-1 \\
0 
\end{bmatrix}
$$

이걸 두 기저벡터가 들어있는 행렬로써 표현하면,

$$

\bold A\vec v =\begin{bmatrix}
0  & -1\\
1  & 0
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
-y\\
x
\end{bmatrix} = \bold u
$$

가 될 수 있겠네요.

실험삼아 두 번 돌려봅시다. 

반시계방향은 다음과 같습니다.

$$
\bold {AA}\vec v =\begin{bmatrix}
0  & -1\\
1  & 0
\end{bmatrix}
\begin{bmatrix}
0  & -1\\
1  & 0
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 \begin{bmatrix}
0  & -1\\
1  & 0
\end{bmatrix}
\begin{bmatrix}
-y\\
x
\end{bmatrix} =
\begin{bmatrix}
-x\\
-y
\end{bmatrix}
=
 \bold u
$$

그리고 시계방향은 다음과 같습니다

$$
\bold {AA}\vec v =\begin{bmatrix}
0  & 1\\
-1  & 0
\end{bmatrix}
\begin{bmatrix}
0  & 1\\
-1  & 0
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 \begin{bmatrix}
0  & 1\\
-1  & 0
\end{bmatrix}
\begin{bmatrix}
y\\
-x
\end{bmatrix} =
\begin{bmatrix}
-x\\
-y
\end{bmatrix}
=
 \bold u
$$

즉 시계방향 90$$^{\circ}$$ 은 

$$
\begin{bmatrix}
0 & 1 \\
-1 & 0
\end{bmatrix}
$$

과 같고,

반시계방향 90$$^{\circ}$$은

$$
\begin{bmatrix}
0 & -1 \\
1 & 0
\end{bmatrix}
$$

과 같음을 알 수 있습니다.

### 임의의 각 쎄타에 대한 회전 행렬

오호 대장님 나오셨군요?

$$\vec{e_1}(\vec{right}) = (cos(\theta),sin(\theta))$$

$$\vec{e_2}(\vec{up}) = (-sin(\theta),cos(\theta))$$ 이므로,

두 기저벡터를 표현한다면 다음과 같습니다.

$$
\vec e_1 = \begin{bmatrix}
cos(\theta)  \\
sin(\theta)
\end{bmatrix}
\\
\vec e_2 = \begin{bmatrix}
-sin(\theta) \\
cos(\theta)
\end{bmatrix}
$$

그리고 두 기저벡터를 통한 행렬을 만든다면,

$$

\bold A\vec v =\begin{bmatrix}
cos(\theta) & -sin(\theta) \\
sin(\theta) & cos(\theta)
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
cos(\theta) x - sin(\theta)y \\
sin(\theta) x + cos(\theta)y
\end{bmatrix} = \bold u
$$

계산해봅시다.

$$\theta = 90(degree)$$으로 잡고, 

$$

\bold A\vec v =\begin{bmatrix}
cos(90) & -sin(90) \\
sin(90) & cos(90)
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =\begin{bmatrix}
0 & -1 \\
1 & 0
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
 - y \\
 x 
\end{bmatrix} = \bold u
$$

뭐랑 비슷하지 않나요? 맞습니다. 반시계방향 90$$^\circ$$회전과 같습니다.

따라서 임의의 각 쎄타에 대한 회전행렬은 

$$
\begin{bmatrix}
cos(\theta) & -sin(\theta) \\
sin(\theta) & cos(\theta)
\end{bmatrix}
$$

로 표현할 수 있습니다.

---

### 합성 함수 개념으로 행렬 곱을 사용해 삼각함수 합의 공식과 배각 공식을 유도하시오.

삼각함수의 합과 배각, 반각에 대해 아는것이 없어 이거 먼저 공부하고 작성하겠습니다.

삼각함수의 합공식은 결국 최종적으로 나오는 값이 무엇인가에 대한 두 함수의 대응 관계입니다.
회전에서 다룬것을 생각해 본다면, 회전이 두번 일어나는 것으로 볼 수 있습니다.

따라서 이는 합성함수로 볼 수 있으며, 합성함수며 실수체계안에 있다면 행렬을 사용해도 무관하겠죠.

$$sin(\alpha + \beta)$$ 라는건, 단위원에서 $$\alpha$$만큼 회전한 다음, $$\beta$$만큼 추가로 회전한다고 볼 수 있습니다.

따라서 회전행렬을 두개 만든 다음, 일단 두 각($$\alpha,\beta$$)을 넣어봅시다.

$$
\bold{R_\alpha}\bold{R_\beta} = 

\begin{bmatrix} 
cos\alpha & -sin\alpha \\
sin\alpha & cos\alpha 
\end{bmatrix} 
\begin{bmatrix} 
cos\beta & -sin\beta \\
sin\beta & cos\beta 
\end{bmatrix} =\\

\\\\\\\\\\\\
\begin{bmatrix} 
cos\alpha cos\beta + 
-sin\alpha sin\beta & 

-sin\beta cos\alpha  +
-sin\alpha cos\beta  \\

sin\alpha cos\beta + 
cos\alpha sin\beta &

-sin\beta sin\alpha + 
cos\alpha cos\beta
\end{bmatrix} = \bold R_{\alpha + \beta}
$$

영역을 매번 $$a_{11},a_{10}$$ 이렇게 적긴 귀찮으니 간단하게 다음 문자에 매핑한다고 가정합시다.

$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
$$

여기서 재미있는게 있습니다.

$$\bold {R_{\alpha}}$$의 c와 $$\bold {R_\beta}$$의 $$c$$ 를 보세요. 나온값이 무엇인가요? 

$$sin(\alpha + \beta) = sin\alpha cos\beta + cos\alpha sin\beta$$ 와 동일합니다.

$$\bold {R_{\alpha}}$$의 c와 $$\bold {R_\beta}$$의 $$a,d$$를 보면, 체의 공리상 덧셈의 교환법칙 및 곱셈의 교환법칙에 의거해 
 $$cos(\alpha + \beta) = cos\alpha cos\beta - sin\alpha sin\beta$$ 와 동일함을 볼 수 있습니다.

$$\bold {R_{\alpha}}$$의 c와 $$\bold {R_\beta}$$의 $$b$$ 는 약간 생각해야합니다.

$$-sin\alpha$$는 $$sin$$함수의 특징인 기대칭(odd symmetry)을 통해 $$sin(-\alpha)$$라고 볼 수 있습니다.

또한 $$-sin\beta$$ 역시 $$sin(-\beta)$$로 볼 수 있죠. 따라서 정리하면 

$$sin(-\alpha + -\beta) = sin(-\alpha)cos(-\beta) + cos(-\alpha) sin(-\beta)$$이며, 

다시 기대칭 성질 및 곱셈의 교환법칙을 통해 $$-sin\alpha cos\beta + -sin\beta cos\alpha$$ 로 나타낼 수 있고 

이는 덧셈의 교환법칙을 통해 $$-sin\beta cos\alpha +-sin\alpha cos\beta$$ 로 나타낼 수 있습니다.

즉 이 역시 위에 설명한 $$c$$케이스와 같음을 볼 수 있습니다.

정리해볼까요?

$$
sin(\alpha + \beta) = sin\alpha cos\beta + cos\alpha sin\beta \\
sin(\alpha - \beta) = sin\alpha cos\beta - cos\alpha sin\beta

\\ \space \\

cos(\alpha + \beta) = cos\alpha cos\beta - sin\alpha sin\beta \\
cos(\alpha - \beta) = cos\alpha cos\beta + sin\alpha sin\beta \\
$$

즉 개념적으로 회전에 대한 덧셈은 행렬곱의 각 원소의 매핑과 같다고 봅니다.

그러면 이제 배각공식을 알아봅시다.

배각, 즉 2배를 해야하니 $$sin(\alpha + \beta) = sin\alpha cos\beta + cos\alpha sin\beta$$ 에서 $$\alpha = \beta$$ 인 경우입니다.

$$
\bold{R_\alpha}\bold{R_\alpha} = 

\begin{bmatrix} 
cos\alpha & -sin\alpha \\
sin\alpha & cos\alpha 
\end{bmatrix} 
\begin{bmatrix} 
cos\alpha & -sin\alpha \\
sin\alpha & cos\alpha 
\end{bmatrix} =\\

\\\\\\\\\\\\
\begin{bmatrix} 
cos\alpha cos\alpha + 
-sin\alpha sin\alpha & 

-sin\alpha cos\alpha  +
-sin\alpha cos\alpha  \\

sin\alpha cos\alpha + 
cos\alpha sin\alpha &

-sin\alpha sin\alpha + 
cos\alpha cos\alpha
\end{bmatrix} = \bold R_{\alpha + \alpha}
$$

앗... 너무 더럽군요 하지만 뽑아봅시다. 

$$c$$위치와 $$d$$위치를 봅시다.

어짜피 $$sin\alpha cos\alpha$$ 는 곱셈의 교환법칙을 만족하는 체계 내에 있으므로, 

$$cos\alpha sin\alpha$$로 볼 수 있습니다.

정리해볼까요?

$$
sin(2\alpha) = sin(\alpha + \alpha) = sin\alpha cos\alpha + cos\alpha sin\alpha = 2sin\alpha cos\alpha \\

cos(2\alpha) = cos(\alpha + \alpha) = cos\alpha cos\alpha - sin\alpha sin\alpha  = cos^2\alpha - sin^2\alpha\\

$$

이렇게 삼각함수의 배각공식을 정리해 보았습니다.

---

### OpenGL은 행기반 행렬을 사용하고 DirectX는 열기반 행렬을 사용한다. 크기 행렬(S), 회전 행렬(R), 이동 행렬(T)가 주어졌을 때 두 시스템에서 사용할 행렬 곱을 어떻게 배치해야 하는지 ( SRT인지 TRS인지 ) 설명하고, 만약 OpenGL에서 사용한 결과를 DirectX로 사용한다면 어떻게 해야할지 정리하시오.

................?

DirectX가 행기반(row major order) 행렬을 사용하며, OpenGL은 열기반(column major order)행렬을 사용하는 것으로 알고있습니다. 

아무튼 그렇다면 DirectX와 OpenGL이 아닌, 행기반(row major order) 행렬 계산과 열기반(column major order) 행렬계산을 기준으로 작성하겠습니다.

row major order와 column major order은 배열에서 어떤게 첫번째로 오는가에 대한 내용입니다.

컴퓨터과학에서 배열을 순회하는 방식이 메모리 공간 지역성에 따른 캐시미스 발생 횟수에 영향을 미치기 때문에 성능에 영향을 줍니다. 그에 따라 구조를 짜내는 것이죠.

열 기반(column major order) 행렬 계산식은 다음과 같습니다.

$$
\bold A_1\vec v_1 =\begin{bmatrix}
a & b\\
c & d
\end{bmatrix}
\begin{bmatrix}
x\\
y
\end{bmatrix} =
 
\begin{bmatrix}
ax+by\\
cx+dy
\end{bmatrix} = \bold u
$$

와 같은 형태로 진행됩니다. 뒤에오는 인자가 변환을 당하는 구조죠.

행 기반(row major order) 행렬 계산식은 다음과 같이 이루어 집니다.

$$
\vec v_2 \bold A_2 =

\begin{bmatrix}
x & y
\end{bmatrix} 
\begin{bmatrix}
a & b\\
c & d
\end{bmatrix}
 =
\begin{bmatrix}
ax+cy & bx+dy
\end{bmatrix} = \bold u
$$

뭔가 재미있지 않나요? 구성이 꽤나 비슷합니다.

전치(Transpose)를 생각해 봅시다

$$
\vec v_1^{T} \bold A_1^T =

\begin{bmatrix}
x & y
\end{bmatrix} 
\begin{bmatrix}
a & c\\
b & d
\end{bmatrix}
 =
\begin{bmatrix}
ax+by & cx+dy
\end{bmatrix} = \bold u
\\
\bold u^T = \begin{bmatrix}
ax+by \\
 cx+dy
\end{bmatrix}
$$

짜잔. 사실 이렇게 됩니다.

즉 행기반과 열기반은 각각의 행렬에 전치를 한 후, 순서만 바꿔서 진행하면 됩니다.
그리고 이건 다른말로 그냥 통째로 묶어서 전치하면 됩니다.

이제 SRT와 TRS의 순서결정이 남았군요.

그 전에 이동 회전 크기 중 어떤것이 먼저 적용되어야 하는가에 대한 문제가 있습니다.

『답은 이동이 가장 마지막에 적용된다』 입니다.

회전은 무엇인가요? 원점을 중심으로 한 기저벡터의 변환입니다.

즉 이동을 먼저한다면, 모든 좌표계가 원점을 중심으로 어느정도 이동한 상태기 때문에

우리가 원하는 것인 오브젝트의 회전이 정상적으로 일어나지 않게 됩니다.

$$\therefore$$ 따라서 열기반(column major order)에서는 $$\bold{TRS} v$$ , 행기반(row major order)에서는 $$v \bold{SRT}$$  순으로 배치해야합니다.

$$\therefore$$다른 기반(major order)을 사용하는 곳에서 사용을 할려면 단순히 전치하면 됩니다.

---

### **행렬의 판별식 ad-bc가 0이 되면 왜 역행렬이 존재하지 못하는지 그 이유를 쓰시오.**

ad-bc = 0 가 되면 두 기저벡터를 변으로 하는 마름모의 넓이가 0이 된다.

마름모의 넓이가 0이라는 뜻은, 두 기저벡터의 각도가 0도라는 뜻이며,

두 기저벡터의 각도가 0이라는 뜻은 두 기저벡터는 선형의존관계라는 뜻입니다.

다시정리하자면 선형의존관계에서는 두 벡터를 통해 새로운 벡터를 생성, 즉 스팬하지 못하기 때문에 행렬의 판별식 ad-bc가 0이 되면 역행렬이 존재하지 못합니다.

---

### 크기 회전을 수행하고 회전 행렬을 수행(RS)하는게 맞는가? 아니면 회전 행렬을 수행하고 크기 회전(SR)을 수행하는 것이 맞는가?

어....단어의 느낌을 통일하기위해 크기변환 및 회전변환 이라고 저는 수정하겠습니다.

일반적으로 생각할 때, 크게 상관없다고 느낄 수 있습니다.

하지만 아주 간단한 예시를 들어보죠. 크기변환이 만약 x축이나 y축에 대하여 음수를 갖게 된다면 어떨까요?

예시를 하나 가져봅시다. 
크기변환이 1,1 → -1,1 인 $$\bold S = 
\begin{bmatrix}
-1 & 0 \\
0 & 1
\end{bmatrix}$$  행렬과,

회전변환의 각($$\theta$$) 이 45도인 $$\bold R = 
\begin{bmatrix}
cos\theta & -sin\theta \\
sin\theta & cos\theta 
\end{bmatrix}$$ 행렬 이

$$\vec{v} = 
\begin{bmatrix}
1\\
0
\end{bmatrix}$$를 변환시키려 합니다.

이하의 케이스는 열기반(column major order) 행렬계산 기준입니다.

 

SR의 케이스를 생각해보죠. 1사분면과 4사분면의 사이, x축에 얹어져있는 v는 

(R)시계방향, 즉 1사분면으로 45도 변환한 다음 

(S)2사분면으로 변환합니다

RS의 케이스를 생각해보죠. 1사분면과 4사분면의 사이, x축에 얹어져있는 v는 

(S)2사분면과 3사분면의 사이, x축으로 변환된 다음,

(R)시계방향, 즉 3사분면으로 45도 변환합니다.

여기서 R의 경우는 크기가 어떠하든 상관없이 시계방향으로 45도 변환하는 것을 알 수 있습니다.

하지만 S는 회전을 언제 하였나에 따라 최종 결과값이 바뀌게 됩니다.

여기서 볼 수 있듯, 크기는 회전의 영향을 받습니다.

그리고 위에서 말했듯 회전은 이동의 영향을 받습니다.

$$\therefore$$ 그렇기 때문에 RS, 이동을 포함한 TRS가 열기반 행렬(column major order) 에서 수행해야 할 옳은 순서 입니다.

---

### 2차원 행렬의 행렬식 ad-bc 값은 변환된 공간의 넓이 변화를 측정하는데 사용된다. 행렬식 값이 양수가 아닌 경우를 찾아보시오. ad-bc가 양수가 아니라면 그 결과가 가지는 의미는 무엇인가?

생각을 좀 해봤습니다. 

두 기저 벡터가 서로를 향해 모이는 것이 넓이가 감소되는 것이라면, 만약 진행방향을 그대로 둔 채 서로 교차하는게 음수가 되는게 아닐까요?

예를 들어 봅시다. $$\vec{e_1} = 
\begin{bmatrix}
 1 \\
 0
\end{bmatrix}$$  과  $$\vec{e_2} = 
\begin{bmatrix}
 0 \\
 1
\end{bmatrix}$$  인 행렬이 

$$\vec{e_1}$$은 90도 만큼 회전하고, $$\vec{e_2}$$는 -90도만큼 회전한다면

$$
\begin{bmatrix}
a & b\\
c & d
\end{bmatrix}
에\space 대하여\\
\begin{bmatrix}
1 & 0\\
0 & 1
\end{bmatrix} 이던\space 공간이 
\begin{bmatrix}
0 & 1\\
1 & 0
\end{bmatrix} 이 된다
$$

즉 공간이 뒤집히게 됩니다. 두 기저벡터가 서로 방향성을 반대로 갖게 되는 것이지요.

이때 ad - bc = 0 - 2 = -2 가 됩니다.

$$\vec{e_1} = 
\begin{bmatrix}
 1 \\
 0
\end{bmatrix}$$  과  $$\vec{e_2} = 
\begin{bmatrix}
 0 \\
 1
\end{bmatrix}$$  가 서로 반대를 갖게 되니, 

각 기저벡터가 선형의존을 지나 기존과는 다른 기저벡터로써 선형독립하게 됩니다.

그리고 저는 이걸 면이 뒤집히는 느낌이라고 생각합니다.

즉 넓이의 부호가 공간이 정방향인지 아닌지를 의미한다고 생각합니다.

혹은 두 기저벡터가 서로가 서로에게 어떤 위치관계를 갖는지를 설명하는 값이라고 생각합니다.

($$\vec{e_2}$$는 $$\vec{e_1}$$의 반시계방향에 위치한다 등)

아마 외적을 배운다면 더 제대로 설명할 수 있지 않을까 생각합니다.

아니 애초에 이게 외적이 아닐까요?