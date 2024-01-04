---
layout: post
title: 풀 지오메트리 쉐이더
katex: True
categories: [Graphic,Shader]
---
2022년 5월 24일

# 기본적 구현

---

이번에는 강의중에 잠시 언급된 GrassShader를 구현해 볼 것입니다.

## 구현물

![Untitled](/assets/(Graphic)Grass/Untitled.png)

![GIF 2022-05-18 오후 2-05-47.gif](/assets/(Graphic)Grass/GIF_2022-05-18_%EC%98%A4%ED%9B%84_2-05-47.gif)

재료는…. 

[https://roystan.net/articles/grass-shader.html](https://roystan.net/articles/grass-shader.html)

[https://catlikecoding.com/unity/tutorials/advanced-rendering/tessellation/](https://catlikecoding.com/unity/tutorials/advanced-rendering/tessellation/)

여기있습니다.

구성은 단순합니다.

## 절차

---

1. 지오메트리 쉐이더를 통해 삼각형 생성하기
2. 색상 및 회전 처리 (행렬곱)
3. 테셀레이션
4. 윈드
5. 바닥점에만 해당하는 바닥맞추기-변환
6. 스트립 형식을 통한 풀의 추가 버텍스 -  곡률 설정
7. 그림자 설정
8. 조명 설정
9. 포그 설정
10. 추가 렌더텍스처를 이용한 상호작용

## 전처리

몇가지 전처리가 필요합니다.

일단 테셀레이션과 지오메트리를 사용하겠다는 명령어가 필요합니다

또한 vertex, hull, domain, geometry,fragment 에 대한 지정이 필요하죠.

```csharp
#pragma require geometry
#pragma require tessellation tessHW

#pragma vertex geomVert
#pragma hull hull
//tessellation
#pragma domain domain
#pragma geometry geom

#pragma fragment frag
```

그리고 헐, 도메인 쉐이더등에 추가 어트리뷰트가 존재하는데,

이는 HLSL, 쉐이더 모델5에 존재하는 어트리뷰트입니다.

![Untitled](/assets/(Graphic)Grass/Untitled%201.png)

패치 유형에 관한 설정을 할 수 있기 때문에,

삼각형으로 만들것이므로 

```hlsl
[domain("tri")]
[outputcontrolpoints(3)]
[outputtopology("triangle_cw")] // *
[partitioning("integer")]
[patchconstantfunc("patchConstantFunc")]
```

> * cw는 clock wise - 시계방향,  
ccw는 counter-clockwise, 반시계방향입니다.
> 

다음과 같이 설정합니다.

토폴로지에서 삼각형을 어떻게 생성할 것인가에 대하며, 쉽게생각하면 앞면과 뒷면에 대한 것 입니다 (backFaceCull이 외적을 통해 앞면과 뒷면을 검사하는것과 유사합니다.)

patchConstantFunc는 후술합니다.

## VertexShading

보통은 버텍스쉐이더에서 미리 위치정보를 월드기반으로 변경하여 작성하지만,

저희는 hull - domain - geometry 를 거쳐야하기때문에 일단 그냥 패스합니다.

최종적으로 만들어진 버텍스 - 도메인 단계에서 

버텍스쉐이더 역할을 해줄 tessVert함수를 통해 변환을 수행할 예정입니다.

만약 이 단계에서 월드좌표로 변경했다면,

플랫폼이나 기기에 따라 좌표가 이상해지는 오류가 발생할 수 있습니다.

```hlsl
VertexOutput geomVert(VertexInput v)
{
    VertexOutput o;

    o.vertex = v.vertex;
    o.normal = v.normal;
    o.tangent = v.tangent;
    o.uv = TRANSFORM_TEX(v.uv, _GrassMap);
    return o;
}
```

## Hull & Tessellation

InputPatch라는 새로운 데이터타입이 있습니다.

제어점 배열을 나타내는데, 간단하게 어떤 버텍스를 뱉을 것 인지만 알려주면 됩니다.

```hlsl
[domain("tri")]
[outputcontrolpoints(3)]
[outputtopology("triangle_cw")]
[partitioning("integer")]
[patchconstantfunc("patchConstantFunc")]
VertexInput hull(InputPatch < VertexInput, 3 > patch, uint id : SV_OutputControlPointID)
{
    return patch[id];
}
```

여기서 패치 상수 데이터를 계산하는식은 다음과 같습니다.

```hlsl
//_ProjectionParams = {1.0 (or –1.0 if currently rendering with a flipped projection matrix), near plane, far plane , 1/FarPlane}
float tessellationEdgeFactor(VertexInput vert0, VertexInput vert1)
{
    float3 vertexPos = TransformObjectToWorld(lerp(vert0.vertex, vert1.vertex, 0.5f));
    float dist = distance(vertexPos.xyz, _WorldSpaceCameraPos);

    //near,far remap=> 0,1 -> 1,0
    float rate = smoothstep(0, _TessellationGrassDistanceFar, dist);
    float inverseLinearizedDistance10 = (1 - rate);
    float steppedFactor = ceil(inverseLinearizedDistance10 * _TessellationGrassDistance) / _TessellationGrassDistance;

    //small is large interval between geom
    return steppedFactor * _TessellationGrassDistanceRate;
}

TessellationFactors patchConstantFunc(InputPatch < VertexInput, 3 > patch)
{
    TessellationFactors f;

    f.edge[0] = tessellationEdgeFactor(patch[1], patch[2]);
    f.edge[1] = tessellationEdgeFactor(patch[2], patch[0]);
    f.edge[2] = tessellationEdgeFactor(patch[0], patch[1]);
    f.inside = (f.edge[0] + f.edge[1] + f.edge[2]) / 3.0f;

    return f;
}
```

카메라와의 거리에 따라 테셀레이션 팩터를 조절하여, cascade처럼 단계를 둔 후, 

카메라 위치 기준 멀리있는 지역의 풀은 그리지 않도록 처리합니다.

## Domain

HullShader에서 받은 제어점의 정보로 보간 작업을 해줘야합니다. 

이때 무게중심좌표를 사용하는데... 

[무게중심 좌표가 가지는 의미와 특징](/posts/11-Convex,-Line-Clipping,Mesh-ebcc20c2d47f493aa66a637b654d8cdf/?pvs=21#%EB%AC%B4%EA%B2%8C%EC%A4%91%EC%8B%AC-%EC%A2%8C%ED%91%9C%EA%B0%80-%EA%B0%80%EC%A7%80%EB%8A%94-%EC%9D%98%EB%AF%B8%EC%99%80-%ED%8A%B9%EC%A7%95)

무게중심좌표는 기존 정리된 내용을 재사용하겠습니다.

아무튼 그렇게 무게중심좌표를 통한 삼각형 안에서의 특정 점의 보간을 위해 

각 버텍스와 노멀, 탄젠트와 UV를 보간합니다.

그 후 해당정보로 버텍스정보를 만들어 넘깁니다.

```hlsl
[domain("tri")]
VertexOutput domain(TessellationFactors factors, OutputPatch < VertexInput, 3 > patch, float3 barycentricCoordinates : SV_DomainLocation)
{
    VertexInput i;

    #define INTERPOLATE(fieldname) i.fieldname = \
            patch[0].fieldname * barycentricCoordinates.x + \
            patch[1].fieldname * barycentricCoordinates.y + \
            patch[2].fieldname * barycentricCoordinates.z;

    INTERPOLATE(vertex)
    INTERPOLATE(normal)
    INTERPOLATE(tangent)
    INTERPOLATE(uv)

    return tessVert(i);
}
```

그 후 Vertex shader에서 수행하던 월드좌표변환을 수행해줍니다.

```hlsl
VertexOutput tessVert(VertexInput v)
{
    VertexOutput o;
    
    o.vertex = float4(TransformObjectToWorld(v.vertex), 1.0f);
    o.normal = TransformObjectToWorldNormal(v.normal);
    o.tangent = float4(TransformObjectToWorldNormal(v.tangent).xyz, v.tangent.w);
    o.uv = v.uv;
    return o;
}
```

## Geometry

여기까지 진행했다면, 버텍스는 성공적으로 생성되었습니다.

그러면 이제 대망의 지오메트리 쉐이더입니다

생각보다 복잡하지만, 단순합니다.

탄젠트 공간을 만들어 로컬과 탄젠트공간을 변환할 수 있는 행렬을 만들고,

풀의 다향성을 주기위한 랜덤회전, 랜덤 꺾임, 바람에 대한 회전행렬을 구성하고

입력값을 기반으로 쌓아올립니다. 대신 strip으로 쌓아올리는데,

최종적으로 RestartStrip을 해서 토폴로지를 변환합니다.

```hlsl
[maxvertexcount(BLADE_SEGMENTS * 2 + 1)]
void geom(point VertexOutput input[1], inout TriangleStream<GeomData> triStream)
{
    float3 grassMap = tex2Dlod(_GrassMap, float4(input[0].uv, 0, 0)).rgb;
    float grassVisibility = grassMap.r;
    float grassOverrideBend = grassMap.g;
    float grassOverrideHeight = grassMap.b;

    if (grassVisibility >= _GrassThreshold)
    {
        float3 pos = input[0].vertex.xyz;
        float3 mspos = TransformWorldToObject(pos);

        float3 normal = input[0].normal;
        float4 tangent = input[0].tangent;
        float3 bitangent = cross(normal, tangent.xyz) * tangent.w;

        float3x3 tangentToLocal = float3x3
        (
            tangent.x, bitangent.x, normal.x,
            tangent.y, bitangent.y, normal.y,
            tangent.z, bitangent.z, normal.z
        );
        tangentToLocal = mul(unity_WorldToObject, tangentToLocal);

        // Rotate around the y-axis a random amount.
        float3x3 randRotMatrix = angleAxis3x3(rand(pos) * UNITY_TWO_PI, float3(0, 0, 1.0f));

        // Rotate around the bottom of the blade a random amount.
        float bend = max(_BendDelta, (1 - grassOverrideBend) * 4);
        float3x3 randBendMatrix = angleAxis3x3(rand(pos.zzx) * bend * UNITY_PI * 0.5f, float3(-1.0f, 0, 0));

        // Wind	
        float2 windUV = pos.xz * _WindMap_ST.xy + _WindMap_ST.zw + normalize(_WindVelocity.xzy) * _WindFrequency * _Time.y;
        float2 windSample = (tex2Dlod(_WindMap, float4(windUV, 0, 0)).xy * 2 - 1) * length(_WindVelocity);

        float3 windAxis = normalize(float3(windSample.x, windSample.y, 0));
        float3x3 windMatrix = angleAxis3x3(UNITY_PI * windSample * grassOverrideBend, windAxis);

        // Transform the grass blades to the correct tangent space.
        float3x3 baseTransformationMatrix = mul(tangentToLocal, randRotMatrix);
        float3x3 tipTransformationMatrix = mul(mul(mul(tangentToLocal, windMatrix), randBendMatrix), randRotMatrix);

        float falloff = smoothstep(_GrassThreshold, _GrassThreshold + _GrassFalloff, grassVisibility);

        float width = lerp(_BladeWidthMin, _BladeWidthMax, rand(pos.xzy) * falloff);
        float height = lerp(_BladeHeightMin, _BladeHeightMax, rand(pos.zyx) * falloff) * grassOverrideHeight;
        float forward = rand(pos.yyz) * _BladeBendDistance;

        float3 tangentNormal = normal;

        // Create blade segments by adding two vertices at once.
        for (int i = 0; i < BLADE_SEGMENTS; ++i)
        {
            float3x3 transformationMatrix = (i == 0) ? baseTransformationMatrix : tipTransformationMatrix;

            float t = i / (float)BLADE_SEGMENTS;
            float temp = t + 1 / (float)BLADE_SEGMENTS;

            //appendix - approx(#1,#2)
            float xpos = ((temp * 1.247993 - 0.101306) - pow(temp * 1.247993 - 0.401306, 4.0) - 0.1) * 0.2 * width;
            float3 offset = float3(xpos, pow(t, _BladeBendCurve) * forward, height * t);
            
            //normal setup for lighting
            float3 surfaceUp = normalize(cross(offset, float3(1, 0, 0)));
            tangentNormal = lerp(tangentNormal, surfaceUp, t);

            //append
            triStream.Append(TransformGeomToClip(mspos, tangentNormal, float3(offset.x, offset.y, offset.z), transformationMatrix, float2(0, t)));
            triStream.Append(TransformGeomToClip(mspos, tangentNormal, float3(-offset.x, offset.y, offset.z), transformationMatrix, float2(1, t)));
        }

        // Add the final vertex at the tip of the grass blade.
        triStream.Append(TransformGeomToClip(mspos, tangentNormal, float3(0, forward, height), tipTransformationMatrix, float2(0.5, 1)));

        triStream.RestartStrip();
    }
}
```

```hlsl
GeomData TransformGeomToClip(float3 pos, float3 normal, float3 offset, float3x3 transformationMatrix, float2 uv)
{
    GeomData o;

    o.pos = TransformObjectToHClip(pos + mul(transformationMatrix, offset));
    o.normal = -TransformObjectToWorldNormal(mul(transformationMatrix, normal));
    o.uv = uv;
    o.worldPos = TransformObjectToWorld(pos + mul(transformationMatrix, offset));
    o.fogCoord = ComputeFogFactor(o.pos.z);
    return o;
}
```

이 코드는 이것저것 많이 추가되어서 많아보이는거지,

실제론 별로 없습니다. 공간변환만 유의해서 진행한다면 크게 어렵지 않습니다.

참고로 저 approx(#1,#2) 부분은 풀의 형태를 수식으로 만든 후, 근사하여 상수로 변경한 것입니다.

부록을 참조해주세요

# 추가구현

## 빛과 그림자

### 그림자

![Untitled](/assets/(Graphic)Grass/Untitled%202.png)

일단 멀티컴파일 처리부터 합시다.

![Untitled](/assets/(Graphic)Grass/Untitled%203.png)

_MAIN_LIGHT_SHADOWS 가 정의되어있거나, 

_MAIN_LIGHT_SHADOWS_CASCADE가 정의되어있을 때,

저희는 이제 그림자를 받아와서 적용해야합니다.

![Untitled](/assets/(Graphic)Grass/Untitled%204.png)

![Untitled](/assets/(Graphic)Grass/Untitled%205.png)

하지만 그림자 색상이 야매입니다.

![Untitled](/assets/(Graphic)Grass/Untitled%206.png)

이걸 가져올 수 있는 방법이 있을까요?

Input.hlsl을 뜯어보면 다음과 같은 데이터들이 있습니다.

![Untitled](/assets/(Graphic)Grass/Untitled%207.png)

_SubtractiveShadowColor? 왠지 말이 되는 이름입니다.

![Untitled](/assets/(Graphic)Grass/Untitled%208.png)

Attenuation의 값은 0에 가까울수록 그림자색상이 나와야하므로, lerp의 앞에 넣어봅니다.

![Untitled](/assets/(Graphic)Grass/Untitled%209.png)

굳

Environment Lighting이 안먹습니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2010.png)

![Untitled](/assets/(Graphic)Grass/Untitled%2011.png)

그런데 이렇게넣으면

![GIF 2022-05-18 오후 12-33-24.gif](/assets/(Graphic)Grass/GIF_2022-05-18_%EC%98%A4%ED%9B%84_12-33-24.gif)

뭔가 여전히 문제가 있습니다.

잘생각해보면 이것은 그림자와 관련이 없습니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2012.png)

단순히 더해줍니다.

![GIF 2022-05-18 오후 12-35-38.gif](/assets/(Graphic)Grass/GIF_2022-05-18_%EC%98%A4%ED%9B%84_12-35-38.gif)

_SubtractiveShadowColor 대신 unity_ShadowColor를 사용해줍니다. 

이게 찐퉁인듯 하네요.

![Untitled](/assets/(Graphic)Grass/Untitled%2013.png)

### 빛

GGX나 때려박아 넣읍시다.

```hlsl
void GGX(in GeomData i, out float lambertian, out float specularTerm)
{
    i.normal = normalize(i.normal);

    //half lambert
    Light light = GetMainLight();
    lambertian = saturate(dot(i.normal, light.direction));
    lambertian = lambertian * 0.5f + 0.5f;

    //BPhong
    float3 view = GetWorldSpaceNormalizeViewDir(i.worldPos);            //ShaderVariablesFunctions.hlsl
    float3 halfDirection = SafeNormalize(view + light.direction);       //common.hlsl
    
    float NoH = saturate(dot(halfDirection, i.normal));
    half LoH = half(saturate(dot(light.direction, halfDirection)));
    
    //GGX
    half LoH2 = LoH * LoH;
    half NoH2 = NoH * NoH;
    float R2 = _Roughness * _Roughness;
    
    float d = NoH2 * (R2 - 1) + 1.00001f;
    half d2 = half(d * d);

    specularTerm = R2 / (d2 * max(half(0.1), LoH2) * (_Roughness * 4.0 + 2.0));
}
```

그렇게 만들어진 frag쉐이더는 다음과 같습니다.

```hlsl
float4 frag(GeomData i) : SV_Target
{
    float4 color = tex2D(_BladeTexture, i.uv);

    //shadow
    #if defined(_MAIN_LIGHT_SHADOWS) || defined(_MAIN_LIGHT_SHADOWS_CASCADE)
        VertexPositionInputs vertexInput = (VertexPositionInputs)0;
        vertexInput.positionWS = i.worldPos;
        
        float4 shadowCoord = GetShadowCoord(vertexInput);

        half shadowAttenuation = saturate(MainLightRealtimeShadow(shadowCoord));
        float4 shadowColor = lerp(unity_ShadowColor, 1, shadowAttenuation);
        //unity_ShadowColor _SubtractiveShadowColor
        color *= shadowColor;
        color *= _MainLightColor;
    #endif

    //apply ggx
    float lambertian, specularTerm;
    GGX(i, lambertian, specularTerm);
    color = color * lambertian + specularTerm;
    
    //ambient
    //PerFrameBuffer, _SubtractiveShadowColor,  _GlossyEnvironmentColor
    color += _GlossyEnvironmentColor;
		return color;
}
```

그러면 간단한 라이트가 먹히는 풀이 나옵니다.

![GIF 2024-01-04 오후 3-27-58.gif](/assets/(Graphic)Grass/GIF_2024-01-04_%EC%98%A4%ED%9B%84_3-27-58.gif)

## 포그 적용

아무래도 기존 버텍스-프래그먼트 쉐이더가 아니다 보니 조금 애로사항이 있습니다.

포그의 개념자체는 아주 단순합니다.

픽셀의 거리값을 기준으로 linear 혹은 exp로 최대 최소치를 설정한 0-1의 값을 구해낸 다음,

fogcolor랑 linear interpolation 처리하는 것이죠.

일단 멀티컴파일 처리를 한 후

![Untitled](/assets/(Graphic)Grass/Untitled%2014.png)

포그의 값을 넣을 수 있도록

최종 fragment 단으로 넘어가는 데이터에서 fogCoordination을 추가합니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2015.png)

ComputeFogFactor의 인자는 clipSpace Position Z. 

그러면 포그설정에 따라 0-1의 값으로 변환해줍니다

![Untitled](/assets/(Graphic)Grass/Untitled%2016.png)

```hlsl
GeomData TransformGeomToClip(float3 pos, float3 normal, float3 offset, float3x3 transformationMatrix, float2 uv)
{
    GeomData o;

    o.pos = TransformObjectToHClip(pos + mul(transformationMatrix, offset));
    o.normal = -TransformObjectToWorldNormal(mul(transformationMatrix, normal));
    o.uv = uv;
    o.worldPos = TransformObjectToWorld(pos + mul(transformationMatrix, offset));
    o.fogCoord = ComputeFogFactor(o.pos.z);
    return o;
}
```

내부적으론 

ComputeFogIntensity를 통해 exp 처리한 값으로 처리해주거나 하는데 

Linear는 단순히 값을 반환하며, 이를 Lerp하여 뱉습니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2017.png)

최종적으로 포그를 처리해서 뱉으면 짜잔

![Untitled](/assets/(Graphic)Grass/Untitled%2018.png)

포그가 달렸습니다.

# 상호작용

잠을 자다가 그런생각을 했습니다.

어? 마스크맵이 있으면, 그냥 텍스처하나 그려서 지워지거나, 눌리거나 할 수 있는거 아닌가?

그렇게 머릿속으로 곰곰히 잘 숙성하다가 개발을 해보았습니다.

골자는 다음과 같습니다.

만약 공격이 발생한다면, 해당지점에 raycast를 사용하여 uv좌표를 가져옵니다.

맵에 사용될 렌더텍스처의 uv좌표에 색을 칠합니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2019.png)

간단하게 충돌대상의 uv를 가져와서,

해당uv에 적절하게 색을 칠해줍니다.

uv는 [0~1]의 값을 가지므로,

텍스처 사이즈와 바운드와 트랜스폼의 크기를 가져와서 적당히 잘 섞어줍니다.

픽셀은 1단위니까요.

![Untitled](/assets/(Graphic)Grass/Untitled%2020.png)

그러면 텍스쳐가 그려지고,

![Untitled](/assets/(Graphic)Grass/Untitled%2021.png)

쉐이더에서 처리해줍니다.

![Untitled](/assets/(Graphic)Grass/Untitled%2022.png)

짜잔

![GrassShader.gif](/assets/(Graphic)Grass/GrassShader.gif)

![GIF 2022-05-23 17-18-57.gif](/assets/(Graphic)Grass/GIF_2022-05-23_17-18-57.gif)

대충 예제는 이렇습니다.

```csharp
using System.Collections;
using System.Collections.Generic;
using System;
using UnityEngine;
using System.Linq;

public class GrassTextureInteraction : MonoBehaviour
{
    [SerializeField]
    private List<Transform> HitTarget;

    [SerializeField]
    private List<Transform> BendTargets;

    [SerializeField]
    private List<Transform> HeightSettingTargets;

    [SerializeField]
    private float reloadRuntime = 0.5f;

    [SerializeField]
    private float heightOverrideValue;

    [SerializeField]
    private float radius;

    [SerializeField]
    private MeshFilter grassGroundMeshFilter;

    [SerializeField]
    private RenderTexture renderTexture;

    private Texture2D texture;

    private readonly List<Vector2> HitPoints = new List<Vector2>();
    private readonly List<Vector2> BendPoints = new List<Vector2>();
    private readonly List<Vector2> HeightSettings = new List<Vector2>();

    void Awake()
    {
        texture = new Texture2D(renderTexture.width, renderTexture.height);
    }

    private void OnEnable()
    {
        //ClearTexture();
        StartCoroutine(UpdateLerping());
    }

    IEnumerator UpdateLerping()
    {
        float t = 0;
        var defaultColors = texture.GetPixels32();
        while (t < reloadRuntime)
        {
            LerpTexture(defaultColors, t / reloadRuntime);
            t += Time.deltaTime;
            yield return null;
        }
    }

    private void FixedUpdate()
    {
        HitPoints.Clear();
        BendPoints.Clear();
        HeightSettings.Clear();
        
        foreach (var target in HitTarget)
        {
            var ray = new Ray(target.position, Vector3.down);

            if (Physics.Raycast(ray, out var hit, 1))
            {
                HitPoints.Add(hit.textureCoord);
            }
        }

        foreach(var target in BendTargets)
        {
            var ray = new Ray(target.position, Vector3.down);

            if (Physics.Raycast(ray, out var hit, 1))
            {
                BendPoints.Add(hit.textureCoord);
            }
        }

        foreach (var target in HeightSettingTargets)
        {
            var ray = new Ray(target.position, Vector3.down);

            if (Physics.Raycast(ray, out var hit, 1))
            {
                HeightSettings.Add(hit.textureCoord);
            }
        }

        ApplyToRenderTexture();
    }

    private void LerpTexture(in Color32[] colors, float t)
    {
        var targetColor = Color.white;
        var list = new List<Color32>();

        foreach (var color in colors)
        {
            list.Add(Color.Lerp(color, Color.white, t));
        }

        texture.SetPixels32(list.ToArray());
        texture.Apply();
    }

    private void ClearTexture()
    {
        Color32 fillColor = Color.white;
        var colors = Enumerable.Repeat(fillColor, texture.width * texture.height).ToArray();

        texture.SetPixels32(colors);
        texture.Apply();
    }

    private void ApplyToRenderTexture()
    {
        var bound = grassGroundMeshFilter.mesh.bounds.size;
        Vector2Int factor = new Vector2Int(
            Mathf.RoundToInt(radius * texture.width / (bound.x * grassGroundMeshFilter.transform.lossyScale.x)),
            Mathf.RoundToInt(radius * texture.height / (bound.z * grassGroundMeshFilter.transform.lossyScale.z)));

        PaintColor(factor, HitPoints, Color.red, 0);
        PaintColor(factor, BendPoints, Color.green, 0);
        PaintColor(factor, HeightSettings, Color.blue, heightOverrideValue);

        texture.Apply();

        Graphics.Blit(texture, renderTexture);
    }

    void PaintColor(in Vector2Int factor, in List<Vector2> points, in Color mask, in float value)
    {
        foreach (var point in points)
        {
            var pos = new Vector2Int(Mathf.FloorToInt(point.x * texture.width), Mathf.FloorToInt(point.y * texture.height));

            var xMin = Mathf.Clamp(pos.x - factor.x, 0, texture.width);
            var xMax = Mathf.Clamp(pos.x + factor.x, 0, texture.width);
            var yMin = Mathf.Clamp(pos.y - factor.y, 0, texture.height);
            var yMax = Mathf.Clamp(pos.y + factor.y, 0, texture.height);

            for (int x = xMin; x < xMax; ++x)
            {
                for (int y = yMin; y < yMax; ++y)
                {
                    var color = texture.GetPixel(x, y);

                    color -= color * mask;
                    color += mask * value;

                    texture.SetPixel(x, y, color);
                }
            }
        }
    }
}
```

# 끝

---

코드는 gist에서 추가하거나 수정합니다

아래 gist를 확인해주세요.

[https://gist.github.com/ashuatz/ed87305c0e1bf7b88d2bce139b41dcc8](https://gist.github.com/ashuatz/ed87305c0e1bf7b88d2bce139b41dcc8)

# Appendix

---

## 풀의 형태

![Untitled](/assets/(Graphic)Grass/Untitled%2023.png)

풀의 한쪽 형태를 위와 같은 형태의 식으로 만들기위해 아래의 식을 근사함. 

```hlsl
//appendix

//approximation #1
// y = ((x/13.6)-(x/13.6 - 0.3)^4 - 0.1)*5.3 + 1/( ((x/13.6)-(x/13.6 - 0.3)^4 - 0.1)*5.3)
// substitution => x/13.6 = w
// y = (w - (w - 0.3)^4 - 0.1)5.3 ≈ (w - (w^4 - 4w^30.3 + 6w^20.3^2 - 4w*0.3^3 + 0.3^4) - 0.1)*5.3
// y = (((-(x + 17.513 - 18) + 18) * 0.073529) - (((-(x + 17.513 - 18) + 18) * 0.073529) - 0.3)^4 - 0.1) * 5.3
// y = (-(x - 0.487) * 0.073529 - ((-(x - 0.487) * 0.073529) - 0.3)^4 - 0.1) * 5.3
//...
//approximation #2
//((17.0 * (temp - 0.0812)) * 0.073529 - pow((((17.0 * (temp - 0.0812)) * 0.073529) - 0.3), 4.0) - 0.1) * 0.2 * width
//((temp * 1.247993 - 0.101306) - pow(temp * 1.247993 - 0.401306, 4.0) - 0.1) * 0.2 * width
```

결과물 - 중간은 통통하고 시작은 얇게 만들 수 있음

![Untitled](/assets/(Graphic)Grass/Untitled%2024.png)

## 플랫폼 문제

일단 OpenGLES 3.2에서 동작을 하지 않습니다. (2021년 기준)

불칸과 DX11에서는 잘 동작하고요.

OpenGL ES 3.2도 일단 geometry shader를 지원하는데 말이죠.

![Untitled](/assets/(Graphic)Grass/Untitled%2025.png)