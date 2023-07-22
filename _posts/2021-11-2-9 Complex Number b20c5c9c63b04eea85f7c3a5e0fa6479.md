---
layout: post
title: 9. Complex Number
katex: True
categories: [그래픽 프로그래밍 3D,assignment]
---
Complex Number

# 머리말

---

...저 죽을 것 같으니까 이번엔 좀 간단하게 가겠습니다...

![결국 -업적 :응급실가서 천장사진 찍기- 깼습니다. ](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/FCzdeBJVQAACdcJ.jpg)

결국 -업적 :응급실가서 천장사진 찍기- 깼습니다. 

# **복소수 대수**

---

## 복소수의 구성요소와 덧셈/곱셈 연산에 대해 정리

---

### 복소수(Complex Number)

두 실수의 순서쌍에 의한 수

표현법은 두가지가 있는데, 

**두 실수의 순서쌍 (a,b)**

**실수와 허수의 합 a+bi**

로 표기합니다.

### 복소수의 구성요소

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled.png)

### 허수(imaginary number) 단위

 $$ 

i^2 = -1
 $$ 

### 복소수의 덧셈

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%201.png)

### 복소수의 곱셈

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%202.png)

## 켤레복소수, 노름을 사용해 덧셈과 곱셈의 항등원 정리(?)

---

.....???????????

켤레복소수, 노름을 사용해 ~~덧셈과 곱셈의 항등원 정리~~ **곱셈의 역수 정리** 라고 이해하겠습니다..

왜냐하면 덧셈의 항등원과 곱셈의 항등원은 그렇게까지 안가도 되거든요.

### 덧셈의 항등원 (Additive Identity)

 $$ 
(a+bi)+(c+di)=(a+bi)\\
\therefore c+di=0
 $$ 

### 곱셈의 항등원(Multiplication Identity)

 $$ 
(a+bi)\cdot(c+di)=(a+bi)\\
\therefore c+di=1+0\cdot i=1
 $$ 

항등원이 나왔으면 역원도 나와야겠죠.

### 덧셈의 역원(Additive Inverse)

 $$ 
(a+bi)+(c+di)=0\\
\therefore c+di=-a-bi=-(a+bi)
 $$ 

### 곱셈의 역원(Multiplication Inverse)

하지만 곱셈의 역원은 그렇게 호락호락하지 않습니다.

허수부를 지워줘야하기 때문인데요, 그걸위해 설명해야할 것들이 있습니다.

자 그러면 먼저 두개부터... 

### 켤레복소수 (Complex Conjugate)

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%203.png)

허수부의 크기만 반대로 한 값입니다.

표기법은 두가지가 있는데, 

 $$ 
z^* : \text{물리학 선호} \\ 
\bar{z} \text{ : vinculum,괄선, 순수수학 선호}
 $$ 

전 그냥 수학에서 쓰듯 쓰겠습니다.

 $$ 
z = a+bi \space ,  \bar{z}=a-bi\\
z\cdot \bar{z}=a^2+b^2
 $$ 

### 크기(노름, Norm)

 $$ 

|z|=\sqrt{a^2+b^2}
 $$ 

### 곱셈의 역원(Multiplication Inverse, Reciprocal)

딱봐도 제곱의 덧셈 형태가 나는 두개가 있죠? 네. 그걸 사용합니다.

제곱하면 거나 여나 똑같다 라는걸 사용하는거죠.

 $$ 
\frac{1}{z} = \frac{\bar{z}}{z\cdot \bar{z}} = \frac{\bar{z}}{|z|^2} = \frac{a-bi}{a^2+b^2}
 $$ 

풀어보면, 중학교때 배우던 곱셈공식으로 묶어서 허수부 제곱을 실수화하며 부호를 돌려주면, 짠

 $$ 
\frac{a-bi}{a^2+b^2} \cdot a+bi = \frac{a^2-(bi)^2}{a^2+b^2} = \frac{a^2+b^2}{a^2+b^2} = 1
 $$ 

어때요 참 재미있죠?

## 단위 복소수의 정의, 단위 복소수의 곱셈의 항등원의 형태

---

### 단위복소수 : 크기(노름)이 1인 복소수

즉 기하학의 벡터에서 단위벡터를 생각하면 편합니다.

성질역시 비슷하죠.(Im = imaginary(허수), Re = Real(실수))

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%204.png)

 $$ 
z=(cos\theta, sin\theta),|z|=1 \\
\because cos^2\theta+sin^2\theta=1
 $$ 

### 단위복소수의 곱셈의 항등원

엥? 이건 다른게 없죠. 단위 복소수의 곱셈의 항등원 역시 1입니다.

아까 구했던 곱셈의 역원식에 대입하여 곱해볼까요?

 $$ 

\frac{cos\theta-(sin\theta)i}{cos\theta^2+sin^2\theta} \cdot cos\theta+(sin\theta)i = \frac{cos\theta^2-((sin\theta)i)^2}{cos\theta^2+sin^2\theta} = \frac{cos\theta^2+sin^2\theta}{cos\theta^2+sin^2\theta} = 1

 $$ 

그러므로 단위복소수의 곱셈의 항등원 역시 1이 됩니다.

### 단위복소수의 곱셈의 역원

여기가 재미있어요.

크기(노름)이 1이여야하기에 값은 코사인제곱과 사인제곱의 형태로 나타낼 수 있게 됩니다.

그렇게 분모가 소거(=1)되면 분자, 즉 켤레복소수만 남게 되죠.

 $$ 
\frac{1}{z} = \frac{\bar{z}}{|z|^2} = \frac{cos\theta-sin\theta i}{cos^2\theta+sin^2\theta } = cos\theta-sin\theta i
 $$ 

즉 단위복소수의 곱셈의 역원은 켤레복소수입니다.

# **복소 평면**

---

## 복소수를 시각적으로 어떻게 평면으로 표현하는지 그 방법을 기술하시오

---

수 직선으로 표현한 그 선과 직교하는 하나의 축을 만들어

하나의 평면을 이루는 '복소평면'으로 나타내는 것입니다.

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%205.png)

유클리드공간의 데카르트좌표계(Cartesian Coordinate System)을 생각해본다면 조금 더 쉽죠.

이제 그렇다면 실수부를 x, 허수부를 y로 둬서 순서쌍으로 나타낼 수 있게 됩니다

 $$ 
a+bi = (a,b)
 $$ 

## 복소 평면에서 단위 복소수들의 집합은 어떻게 표현되는지 정리하고, 삼각함수와 각을 사용해 단위 복소수를 표현하시오.

---

간단합니다. 

### 단위복소수의 집합

이건 결국 크기가 1인 값의 집합이므로 원이 되겠죠?

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%206.png)

그러므로

### 삼각함수와 각을 사용한 복소수 표현

은 다음과 같이 정의할 수 있습니다

 $$ 
z = (cos\theta,sin\theta)\\
z=cos\theta + sin\theta i
 $$ 

## 어떤 복소수에 허수단위 i를 네 번 곱하면 원위치로 돌아옴을 정리

---

수식적으로,

 $$ 
i^2 = -1, (i^2)^2 = (-1)^2 = 1
 $$ 

입니다.

이걸 복소평면으로 본다면

 $$ 
1 = 0\degree \space and \space i = 90\degree
 $$ 

그리고 복소수의 곱셈은 회전을 적용시킨다와 같으므로 

 $$ 
i^4 = i \cdot i \cdot i \cdot i  = 360\degree =0\degree
 $$ 

입니다.

## 어떤 복소수와 단위복소수의 곱은 회전 변환이 됨을 정리하시오.

---

어떤복소수 x + yi 가 있을 때, 이것과 단위복소수 cosA+sinBi를 이항연산한다고 가정합니다.

 $$ 
(x,y) \cdot (cos\theta,sin\theta)\\
(xcos\theta-ysin\theta,
xsin\theta+ycos\theta)

 $$ 

그리고 회전변환을 생각해 봅시다.

 $$ 
\begin{bmatrix} 
cos\theta & -sin\theta \\
 sin\theta & cos\theta \end{bmatrix}

\cdot

\begin{bmatrix}
x \\ 
y 
\end{bmatrix} 

= 

\begin{bmatrix}
xcos\theta  -ysin\theta \\
xsin\theta  +ycos\theta 
\end{bmatrix} 
 $$ 

굳!

또한 어떤 단위복소수와 단위복소수의 곱은, 

삼각함수의 덧셈정리에 의해

 $$ 
(cos A+(sin A)i)(cos B + (sin B)i) = \\
(cos A cos B - sin A sin B)+(sinAcosB+cosAsinB)i=\\
cos(A+B) + (sin(A+B))i
 $$ 

라고 볼 수있겠네요. 회전변환과 동일하죠?

## 임의의 복소수간의 곱은 어떤 변환을 만들어내는지 정리

---

단위복소수의 곱셈은 단위벡터를 가진 회전변환과 동일합니다.

그렇다면 단위복소수가 아닌 것은? 

행렬로 친다면 기저벡터가 단위벡터가 아니라고 생각하면 되니,

행렬의 변환을 두 개로 나누어, 회전과 크기의 행렬곱으로 나타낼 수 있음을 생각한다면

즉 임의의 복소수간의 곱 역시 회전 및 스케일링한 것과 같겠죠?

그러면 임의의 복소수 간의 곱은 크기 및 회전 변환입니다.

단위벡터가 아닌 벡터로 구성된 행렬로의 변환처럼요.

## 수의 곱셈이란 어떤 작업을 하는지 자신의 생각을 정리하시오

---

결국 곱셈은 공간의 변화를 적용하는 것입니다.

예전에는 실수에 대한 수 직선만이 존재하여, 그 수에 대한 크기만 적용이 되었다면, 

이제 허수의 수 직선이 추가적으로 존재하게 되어 그 직교에 의한 회전이 발생하게 됩니다.

## 어떤 복소수와 그 켤레 복소수를 곱하면 노름의 제곱이 된다. 시각적으로 어떤 과정을 거쳐 노름의 제곱이 되는지 정리하시오.

---

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%203.png)

 $$ 
let, z\cdot \bar{z} = (a,b)\cdot(a,-b).\\
(a,b)\cdot(a,-b)=(a^2+b^2,-ab+ab)=(a^2+b^2, 0)\\

 $$ 

그림으로 정리하면 아래와 같습니다.

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%207.png)

# **오일러의 공식 #1**

---

...이거결국  $$ e^{jθ} = cos θ + j\space sin θ $$   하고싶어서 하는거죠..?

## 무리수 e (네이피어 수, 오일러 수)

---

오일러 수 혹은 자연상수인 e는 2.71828과 같은 값을 가지고 있는 무리수입니다.

복리 계산에서 많이 들어가죠.

표현방법으로는 두가지가 있는데, 급수를 통한 방법 및 극한을 통한 방법이 있습니다

 $$ 
e = \sum_{n=0}^{\infin} \frac{1}{n!} = 1+\frac{1}{1}+\frac{1}{1\cdot2}+\cdot\cdot\cdot
 $$ 

 $$ 
e =\lim_{n\rightarrow \infin}{\left(1+\frac{1}{n}\right)^n}
 $$ 

## 무리수 e를 도출하기 위한 극한의 개념을 정리하시오.

---

여기서 극한의 개념을 알아보겠습니다. 

극한을 직관적으로 정의하자면 

 $$ 
\lim_{x\rightarrow a}{f(x)} = L
 $$ 

다음과 같은 식에 대해, x가 a에 한없이 가까워질 때, 함수값 f(x)가 어떤 극한값 L이 됨

이라고 볼 수 있습니다.

어떤예시가있을까요? 수렴하는것들이 있고, 롤의 방어력이 있겠네요.

롤의 방어력은 아무리 많이 올린다고 하더라도 100%의 데미지 감소율이상 올라갈 수 없습니다.

혹은 리스크오브레인2 에서의 곰탱이 인형은 1개당 15%확률의 데미지 감소를 갖지만 아무리 많이먹어도 100%를 초과할 수 없습니다. 100%에 수렴한다고 봐야겠죠.

[곰](https://static.wikia.nocookie.net/riskofrain2_gamepedia_en/images/3/39/Tougher_Times.png/revision/latest/scale-to-width-down/24?cb=20200129193210)

곰

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%208.png)

그렇기때문에 어떤 값으로 나아가는 네이피어수(e)에게는 극한을 쓴다면 쉽게 이해할 수 있죠.

그래서 기존의 급수를 극한으로 표현하자면, 다음과 같이 나타낼 수 있는 것입니다.

 $$ 
e =\lim_{n\rightarrow \infin}{\left(1+\frac{1}{n}\right)^n}
 $$ 

## 지수 함수의 성질과 0승과 -승이 가지는 규칙을 정리하시오.

---

지수함수가 무엇인지 부터 봐야겠죠.

 $$ 
f(x) = a^x
 $$ 

변수를 지수로 두는 함수를 지수함수라 합니다.

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%209.png)

### 0승의 규칙

이때, 지수가 0이며, 밑이 0이 아닐 때 1의 값을 갖습니다.

지수가 0이면 0번 곱했다는 뜻이므로 곱셈의 항등원 1을 갖는 것이죠.

 $$ 
a^m\cdot a^0=a^{m+0}=a^m
 $$ 

### -승의 규칙

그리고 지수가 0 미만, 즉 -를 갖게 되는 순간 재미있어집니다.

어 음수번 곱하였다는건, 반대로 적용했다고 생각하니, 역행렬같은 느낌 아닐까요?

그리고 역행렬  $$ A^{-1} $$ 이 기억나시나요? 네 맞습니다. 

 $$ x^{-1} = \frac{1}{x} $$ 인 것입니다...!

 $$ 
if,m>0\\a^m\cdot a^{-m}=a^{m-m}=a^0=1\\
a^{-m} = \frac{1}{a^{m}}
 $$ 

## 1보다 큰 밑을 가지는 자연지수함수  \\( f(x)=e^x \\) 가 가지는 성질요약

---

일단은 일반적인 값을 먼저 설명을 하자면, 

 $$ 
f(0) = 1 \\
f(1) = e\\
\lim_{x\rightarrow -\infin}{f(x)} = 0\\
\lim_{x\rightarrow \infin}{f(x)} = \infin\\

 $$ 

입니다.

하지만 지수 함수에서 재미있는 부분은 저런게 아니라,

지수함수의 도함수가 함수의 값(높이)과 똑같다는 부분인데요,

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%2010.png)

임의의 점에서 접선을 긋고, 접선이 x축과 닿는 길이가 언제나 1이라는 소리입니다.

그러나 이걸 알려면 접선과 미분, 도함수를 알아야겠군요...............

## 극한의 개념을 사용한 접선의 개념을 설명

---

![Untitled](/assets/9%20Complex%20Number%20b20c5c9c63b04eea85f7c3a5e0fa6479/Untitled%2011.png)

### 접선

일반적으로 접선은 곡선과 직선이 한 점에서 만날 때, 

이 점을 접점이라 하며 동시에 곡선에 접하는 직선을 **접선**이라고 합니다.

### 할선

또한 곡선과 직선이 두 점에서 만날 때,

곡선을 자르게 되는 이 직선을 곡선의 할선이라고 합니다.

그런데 만약 이 할선의 두 점의 변위를 만약 최대한 줄인다면 어떻게 될까요?

이걸 수학적으로 말해본다면, 기울기 m, 임의의 값 a에 대해

 $$ 
let, h = displacement(point_1,point_2)\\
m = \lim_{h\rightarrow 0}\frac{f(a+h)-f(a)}{h} \\
 $$ 

라고 볼 수 있습니다.

따라서 극한의 개념으로 본다면 접선은 두 점의 거리가 0인, 할선이라고 봐도 무방하겠죠.

## 미분 계수와 도함수의 차이에 대해 설명하시오.

---

### 미분계수

이건 저희가 초등학교때부터 해왔던 거리 - 속력 - 시간 관계 및 가속력을 생각하면 편합니다.

거리를 아주 짧은 시간단위로 쪼개면, 즉 미분하면 속력이며, 속력을 미분하면 가속력이 나오죠.

즉 미분계수는 평균 변화율에서 어느지점으로 접근할 때의 순간변화율, 접선의 기울기라고 볼 수 있습니다.

### 도함수

도함수는 각 점에서의 접선의 기울기 즉, 미분계수를 매 위치마다 순간 변화율로 보고, 이를 함수로써 나타낸 것을 말합니다.

수학적으로 말하면 미분계수가 존재하는 모든 점을 정의역으로 하는 함수죠.

원소와 집합의 개념이라고 생각합니다.

# 끝내며

---

다음 주는 수술입니다.

과연 저는 다음 주에도 수학 과제를 해낼 수 있을까요?