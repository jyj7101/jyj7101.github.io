---
layout: post
title: 이끼 버텍스 블렌드 쉐이더 제작
katex: True
categories: [Graphic,Shader]
---

졸업작품 프로젝트 팀을 들어가니 요청을 받았습니다.

> 현재 유적이나 돌 모듈 등에 이끼 표현이 들어가는데, 자연스럽게 표현할 수 있는 쉐이더가 필요합니다. <br/>이 과정에서 필요한 머테리얼이 있다면 요청해주시면 감사하겠습니다.
> 

# 결과물

![Day2 작업물](/assets/MosssVertexColorBlend/mossBlendPaint.gif)
*Day 2 작업물입니다*

# 요청사항 분석

해당 정보로는 정확히 요청사항 분석이 어려웠습니다.

> 현재 유적이나 돌 모듈 등에 이끼 표현이 들어가는데
> 

에서 

업무에 관련한 사항으로

1. 자동화를 하고싶은건가요
2. 대체해야하는건가요?
3. 기능의 추가가 필요한것인가요?

구현에 관련한 사항으로

1. noise가 붙은 stochaostic texture blending을 원하는것인가요?(기능추가)
2. 아니면 일괄적으로 적용할 수 있는 쌓인눈 쉐이더 같은걸 이끼버전으로 원하는건가요?(자동화)
3. 아니면 그냥 레이어드 블렌딩이 필요한건가요?(대체)
4. 두께감이 필요한가요? (=버텍스 처리를 해야하나요?, 기능추가)
5. 자연스럽게 표현할 수 있는 쉐이더. 라면 조명연산하에 음지에 있는 표면에만 나타내어야 하는건가요?(기능추가)

를 추가로 질의했고,

> 아마 버텍스페인팅이나 앞에서말한 텍스쳐맵같은걸로 머트리얼을 블렌하는걸 원하시는게 아닐까 싶습니다..
> 

라는 답변을 받았습니다.

따라서 전부 구현하기로 합니다.

# Stochastic

반복되는 패턴이 최대한 눈에띄지않도록 블렌딩하는 방법입니다.

참조 : [https://blog.unity.com/engine-platform/procedural-stochastic-texturing-in-unity](https://blog.unity.com/engine-platform/procedural-stochastic-texturing-in-unity)

기존에 작성해둔걸 슬쩍 가져옵니다.

4방향 타일링입니다.

```glsl

float4 hash4(in float2 input)
{
    const float2 offset = float2(26.0, 161.0);
    const float domain = 71.0;
    const float invDomain = 1 / domain;
    const float someLargeFixed = 951.135664;
    const float invSomeLargeFixed = (1 / someLargeFixed);

    float4 p = float4(input.xy, input.xy + 1);
    p = frac(p * invDomain) * domain;
    p += offset.xyxy;
    p *= p;
    return frac(p.xzxz * p.yyww * invSomeLargeFixed);
}

void textureTiling1_float(in Texture2D tex, in SamplerState ss, in float2 uv, out float3 color)
{
    float2 p = float2(floor(uv));
    float2 f = frac(uv);

    // generate per-tile transform
    float4 ofa = hash4(p + float2(0, 0));
    float4 ofb = hash4(p + float2(1, 0));
    float4 ofc = hash4(p + float2(0, 1));
    float4 ofd = hash4(p + float2(1, 1));
    
    float2 _ddx = ddx(uv);
    float2 _ddy = ddy(uv);

    // transform per-tile uvs
    ofa.zw = sign(ofa.zw - 0.5);
    ofb.zw = sign(ofb.zw - 0.5);
    ofc.zw = sign(ofc.zw - 0.5);
    ofd.zw = sign(ofd.zw - 0.5);
    
    // uv's, and derivatives (for correct mipmapping)
    float2 uva = uv * ofa.zw + ofa.xy;
    float2 ddxa = _ddx * ofa.zw;
    float2 ddya = _ddy * ofa.zw;

    float2 uvb = uv * ofb.zw + ofb.xy;
    float2 ddxb = _ddx * ofb.zw;
    float2 ddyb = _ddy * ofb.zw;

    float2 uvc = uv * ofc.zw + ofc.xy;
    float2 ddxc = _ddx * ofc.zw;
    float2 ddyc = _ddy * ofc.zw;

    float2 uvd = uv * ofd.zw + ofd.xy;
    float2 ddxd = _ddx * ofd.zw;
    float2 ddyd = _ddy * ofd.zw;
    
    // fetch and blend
    float2 b = smoothstep(0.25, 0.75, f);
    
    color = lerp(
        lerp(tex.SampleGrad(ss,uva, ddxa, ddya), tex.SampleGrad(ss,uvb, ddxb, ddyb), b.x),
        lerp(tex.SampleGrad(ss,uvc, ddxc, ddyc), tex.SampleGrad(ss,uvd, ddxd, ddyd), b.x),
        b.y).rgb;
}
```

노션에서 작성한 파일이 다 날라가서 지금은 잔해밖에 남지않은 코드더미를 사용했습니다.

이것말고도 여러 방식이 존재하는데(보로노이,노이즈텍스처 사용 등),

이번에는 일단 PoC를 위해 제일 간단한 stochastic을 사용했습니다.

![Untitled](/assets/MosssVertexColorBlend/Untitled.png)

타일링 50값 치곤 적당히 잘 된 것 같네요. (Day 1 )

참조 : [https://iquilezles.org/articles/texturerepetition/?fbclid=IwAR1DP44yF51aAJZCZizV1v1GfTDciW_XfOYokIxtybMmCWGeKcQn_v-Pxvo](https://iquilezles.org/articles/texturerepetition/?fbclid=IwAR1DP44yF51aAJZCZizV1v1GfTDciW_XfOYokIxtybMmCWGeKcQn_v-Pxvo)

# 텍스처 블렌드

버텍스컬러를 사용해서 텍스처를 블렌딩해야합니다.

특수하게도, 이번 프로젝트는 기본 버텍스컬러를 검정으로 잡고 작업을 진행합니다.

사실 블렌드 자체는 간단합니다 약식으로 lerp(main,sub,vertexColor.r) 하면 끝이잖아요?

다만, 여기서 구현할 때 많은 수정사항이 존재했는데,

1. 경계부분이 그라데이션으로 풀리는게 아니라 디테일 형태로 남아야함
2. 대신 내부엔 노이즈가 들어가면 안됨
3. 디테일의 형태가 나타나는 영역을 조절할 수 있어야함

3번의 피드백을 통해 수정되었습니다.

## 빛 연관

텍스처에서 월드노말을 가져와, 빛 * -1 과 내적합니다. 이를 OneMinus 하면 빛과 바라보는방향이 같을때 0을 뱉고, 전혀다를때 2를, 직교할때 1을 뱉습니다.

이걸 이제 SunDirOffset이라는 오프셋 파라메터를 통해 더해주고 saturate하여 빛과 이끼의 상호작용을 구현합니다. 

## 노이즈 블렌드

일단 노이즈 텍스처와 알파 문턱값을 추가하고, 알파값을 받아오도록 합니다.

노이즈가 마음에 안들수있으니 인텐시티를 추가로 노이즈에 곱하고,

### 알파가 문턱값을 넘길때 - A

알파 문턱값과 노이즈 텍스처를 step, 영역을 확보합니다.

이를 알파값과 곱해 문턱보다 알파가 높을때, 하얀색이 나올 수 있도록 처리합니다

### 알파가 문턱값을 못넘길때 - B

어느정도까지 못넘기는지 선형적인 느낌으로 표현하기 위해 

smoothStep을 최대값을 문턱, in값을 알파로 설정합니다. 

저 위 둘(A,B) 을 이제 블렌드(lerp) 한 다음, 

전체적으로 어두워졌기때문에 알파값을 최대치로한 리맵핑을 적용합니다.

끄트머리 처리

알파 문턱값을 제곱한 값을 노이즈 * 인텐시티 값과 또 step해줍니다.

블렌드한 값을 다시 A와 블렌드(Lerp)하되, 이때 위의 step값을 사용해주면 

끄트머리에 남는 디테일들은 알파값이 동일하게 적용되어 ‘짜잘하게 남는’ 디테일이 완성됩니다.

![Untitled](/assets/MosssVertexColorBlend/Untitled%201.png)

![Day 2 문턱값과 세기를 통한 조절 ](/assets/MosssVertexColorBlend/mossBlendparam.gif)
*Day2-문턱값과 세기를 통한 조절*

# 버텍스 모핑(Morphing)

그럼 이제 두께감을 줘봅시다.

일단 간단한 방법으로 버텍스쉐이더단에서 버텍스 노말 방향으로 버텍스포지션을 수정하는 것입니다.

대신 이 경우, 중첩된 버텍스가 각 다른 버텍스컬러를 가질 경우(면을 채색할 경우),

아웃라인이 깨지듯 그냥 균열이 일어날 수 있습니다.

하지만 우리의 아티스트는 잘 칠해줄 것이라 믿고 (현재 로우폴밖에 없어서 테스트가 힘듦)

2일차 작업을 마칩니다.

![Day2 칠하는것 까지 확인](/assets/MosssVertexColorBlend/mossBlendPaint.gif)
*Day2-칠하는것 까지 확인*