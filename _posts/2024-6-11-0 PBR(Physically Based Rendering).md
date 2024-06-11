---
layout: post
title: PBR(Physically Based Rendering)
katex: True
categories: [Graphic]
---

# PBR(Physically Based Rendering) 개요

PBR(Physically Based Rendering)은 현실 세계의 물리적 법칙을 기반으로 이미지를 렌더링하는 기술입니다.

PBR의 목표는 현실적이고 일관된 결과를 제공하여 시각적으로 설득력 있는 이미지를 생성하는 것입니다.

이를 위해 PBR은 표면의 물리적 특성을 수학적으로 모델링합니다.

<br/>

이것을 이제 요소와 개념으로 나눠서 보겠습니다.

# PBR의 주요 요소

PBR은 다음의 주요 요소들을 포함하여 물체의 시각적 특성을 정의합니다.
유니티나 언리얼 재질요소에서 자주보던 친구들입니다.

1. **알베도(Albedo)**:
    - 물체의 기본 색상을 나타내며, 반사나 투과의 영향을 받지 않는 물체 자체의 색상입니다. <br/>
    이를 통해 물체의 기본적인 색감을 정의할 수 있습니다.

2. **노멀 맵(Normal Map)**:
    - 표면의 세부적인 기하학적 특성을 정의하는 데 사용됩니다. <br/>
    작은 디테일이나 질감을 표현할 수 있도록 도와주며, 빛이 표면에 어떻게 반사되는지를 결정합니다.

3. **메탈릭(Metalness)**:
    - 물체의 금속성을 나타냅니다. 값이 0에 가까울수록 비금속 재질을, 1에 가까울수록 금속 재질을 의미합니다.<br/>
    금속 재질은 빛을 반사하는 특성이 다릅니다.

4. **거칠기(Roughness)**:
    - 표면의 매끄러움 정도를 나타내며, 빛이 어떻게 확산되는지를 결정합니다. <br/>
    표면이 매끄러울수록 반사가 명확하고, 거칠수록 빛이 더 많이 퍼집니다.

5. **환경 반사(Environment Reflection)**:
    - 물체가 주변 환경에서 빛을 어떻게 반사하는지를 정의합니다. <br/>
    환경 맵(Environment Map)을 사용하여 물체 표면에 반사되는 주변 이미지를 정확하게 표현할 수 있습니다.

# PBR의 주요 개념

PBR의 효과적이고 물리적으로 정확한 렌더링을 위해 다음의 주요 개념이 사용됩니다

## 반사 관련

1. **미세 표면 분포 (Microfacet Distribution)**:<br/>
    **Specular Reflection**

    정반사, 스펙큘러 하이라이트에 대한 여러 모델입니다.
    
    - **베크만 분포(Beckmann Distribution)**:초기의 Microfacet 모델 중 하나로, 표면의 미세 기울기를 가우시안 분포로 가정합니다. 스펙큘러 하이라이트가 더 집중된 형태로 나타납니다.
    - **Phong**:초창기에 사용되던 모델들로,하이라이트가 매끄럽고 작은 영역에 집중되는 특성을 가지고 있습니다.
    - **Blinn-Phong**: 퐁모델을 수정한 버전으로, 연산최적화가 진행되었습니다. 스펙큘러 하이라이트가 더 부드럽습니다.
    - **Cook-Torrance**:미세 기울기의 통계적 분포를 기반으로 하는 모델입니다. 이는 실제 물리적 표면의 반사를 잘 표현하며, 에너지 보존을 충실히 따릅니다.<br/>
    이후 GGX 분포가 개발되며 D식이 GGX 분포를 사용하기 시작합니다. 자세한건 양방향 반사도 분포 함수의 아래식 참조.
    - **GGX (Trowbridge-Reitz Distribution)**: 쿡 토랜스 모델의 최적화 버전이며, 표면의 거칠기를 표현하는 미세 표면 분포의 한 종류로, 거친 표면에서 빛의 분산을 잘 표현합니다.
    - **Schlick-GGX**:GGX 분포를 기반으로 한 슈릭-GGX는 반사광의 스무딩을 더 자연스럽게 하기 위한 모델입니다. 프레넬 효과와 결합하여 더욱 사실적인 반사를 구현할 수 있습니다. [Schlick 근사](https://en.wikipedia.org/wiki/Schlick%27s_approximation)

2. **양방향 반사도 분포 함수 (BRDF, Bidirectional Reflectance Distribution Function)**: <br/>
    표면의 특정 지점에서 들어오는 빛과 반사되는 빛의 관계를 정의하는 함수입니다. <br/>
    BRDF는 표면의 특정 지점에서 입사광과 반사광의 방향에 따라 반사되는 빛의 양을 정의합니다.<br/>
    
    
    BRDF는 다음의 네 가지 변수를 고려하여 정의됩니다<br/>
    1. 입사방향 $$ \omega_i $$
    2. 반사방향 $$ \omega_r $$
    3. 법선벡터 $$ N $$
    4. 반사강도 
    
    예시론 다음과 같은 모델이 있습니다.

    **Diffuse Reflection**

    - **Lambertian Model**: 램버트 모델은 가장 단순한 형태의 BRDF로, 표면에서 빛이 모든 방향으로 균일하게 반사되는 이상적인 확산 반사를 가정합니다. 이는 매트(건조)한 표면을 표현하는 데 적합합니다. 

    - **half-Lambert Model**: 밸브의 하프라이프에서 사용해서 유명한 그 식입니다. <br/>
    값을 단순하게 곱하고 더해 리맵핑해서, 표면이 빛을 받지 않는 영역에서도 완전히 어두워지지 않도록 해, <br/>
    자연스러운 조명 효과를 제공합니다.

    - **오렌 나이어(Oren-Nayar Model)**:오렌-나이어 모델은 거친 확산 반사를 설명하는 BRDF로, 표면의 거칠기를 고려하여 다양한 각도에서의 반사를 모델링합니다.<br/>
    이는 매트한 표면에서의 반사를 더욱 현실적으로 표현하며, 램버트를 대체하기위해 레퍼런스 그래프를 먼저 그리고 그 그래프를 구현한 것에 가깝습니다. 식이 복잡하단 뜻이죠.

    **Specular Reflection**

    - **쿡 토랜스 모델(Cook-Torrance Model)**:쿡-토랜스 모델은 미세 표면 분포와 프레넬 반사를 포함하여 표면의 거칠기와 금속성을 고려한 복합적인 BRDF입니다.<br/>
    금속성과 비금속성 표면을 모두 잘 표현할 수 있습니다.아래 식에 일단 들어갈건 다 들어갔죠? (D = 마이크로페이싯 분포 함수, F = 프레넬, G = 기하학감쇠,미세면감쇠)

    $$ f_r(\omega_i, \omega_r) = \frac{D(h) \cdot F(\omega_i, h) \cdot G(\omega_i, \omega_r, h)}{4 (\omega_i \cdot N) (\omega_r \cdot N)} $$ <br/>
    
    - **워드 모델 (Ward Model)**:워드 모델은 이방성 반사를 설명하는 BRDF로, 표면의 특정 방향으로 빛이 더 많이 반사되는 경우를 모델링합니다. 이는 브러시드 메탈이나 머리카락 같은 재질을 표현하는 데 적합합니다.<br/>
   
     $$ f_r(\omega_i, \omega_r) = \frac{1}{4 \pi \alpha_x \alpha_y \sqrt{(\omega_i \cdot N)(\omega_r \cdot N)}} \exp \left( - \left( \frac{(\omega_i \cdot H)^2}{\alpha_x^2} + \frac{(\omega_r \cdot H)^2}{\alpha_y^2} \right) \right) $$

    **Diffuse Reflection + Specular Reflection**

    - **디즈니 BRDF(Heitz 모델)**: 디즈니 BRDF는 다양한 표면 특성을 단일 모델로 통합하여, 사용자가 조정할 수 있는 여러 매개변수를 제공합니다.<br/> 이는 다양한 재질을 하나의 모델로 표현할 수 있도록 합니다. 디퓨즈는 오렌-나이어모델을, 스펙큘러는 쿡-토랜스 모델을 사용합니다.

    [Physically Based Shading at Disney](https://media.disneyanimation.com/uploads/production/publication_asset/48/asset/s2012_pbs_disney_brdf_notes_v3.pdf)

    [Rendering the Moana Island Scene Part 1: Implementing the Disney BSDF](https://schuttejoe.github.io/post/disneybsdf/)

    [Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs](https://jcgt.org/published/0003/02/03/)


3. **프레넬 반사 (Fresnel Reflectance)**:
    - 빛이 물체 표면에 입사할 때, 입사각에 따라 반사되는 빛의 양이 변하는 현상을 설명합니다. 물체의 표면에서 빛이 더 평행하게 입사할수록 반사가 강해집니다.<br/>
    [Schlick 근사](https://en.wikipedia.org/wiki/Schlick%27s_approximation)를 보통 사용하며, 최근?[Bringing an Accurate Fresnel to Real-Time Rendering: a Preintegrable Decomposition](https://belcour.github.io/blog/research/publication/2020/08/26/brdf-fresnel-decompo.html)도 나왔습니다.<br/>
    Belcour와 다른 저자들은 프레넬 반사와 BRDF를 사전 통합(preintegrable) 방식으로 분해하여 복잡한 반사 모델을 실시간으로 효과적으로 구현할 수 있게 했습니다.

4. **지향성 반사 (Directional Reflectance)**:
     - 지향성 반사는 특정 방향에서 빛이 반사되는 정도를 설명합니다. <br/>
     이는 표면의 방향성과 관련이 있으며, 광택이 있는 표면에서 특히 중요한 역할을 합니다. <br/>
     이방성,미세 표면 분포이 포함되어있습니다.

## 조명 관련

5. **에너지 보존 (Energy Conservation)**:
    - 표면에서 반사되는 빛의 양이 입사하는 빛의 양을 초과할 수 없음을 의미합니다.<br/>
    당연하게도 모든 물리 기반 모델은 에너지 보존을 준수해야 합니다.

6. **이방성 (Anisotropy)**:
    - Anisotropy는 물체 표면에서 빛이 특정 방향으로 더 많이 반사되는 현상을 의미합니다.<br/>
    이는 표면의 미세 구조가 특정 방향으로 정렬되어 있을 때 발생하며,머리카락, 브러시드 메탈, 직물 등에서 주로 나타납니다.<br/>
    이러한 특성을 현실감 있게 모델링하기 위해 PBR에서 Anisotropy는 중요한 역할을 합니다.<br/>
    아까의 워드 모델이 이 이방성모델에 해당됩니다.

7. **헴리스페릭 라이트(Hemispheric Light)**:
     - 헴리스페릭 라이트는 하늘과 지면에서 오는 빛을 모델링하여, 간접 조명을 보다 사실적으로 표현합니다.<br/>
     특히 외부 환경에서의 조명 효과를 개선하는 데 유용합니다.

8. **차분광 스펙트럼 (Spectral Rendering)**:
     - 차분광 스펙트럼은 빛의 파장에 따라 다르게 반사되고 굴절되는 물체의 특성을 설명합니다.<br/>
     다양한 색상과 재질을 보다 정확하게 표현하는 데 사용됩니다.

## 필터링 관련

9. **입사광 필터링 (Incoming Light Filtering)**:
    - 표면에 도달하는 빛이 표면의 미세 구조에 의해 어떻게 필터링되는지를 설명합니다. <br/>
    거친 표면에서의 빛의 확산과 반사를 보다 정확하게 모델링하며,이전에 소개한 마이크로페이싯 모델과 AO, 그림자맵들을 통해 구현됩니다...

## 그림자 관련    

10. **주변광 차폐(Ambient Occlusion, AO)**:
    - AO는 물체의 표면에서 주변 환경에 의해 차단된 빛의 영향을 계산합니다. 이는 특히 모서리나 구석진 부분에서 그림자가 더 진하게 보이도록 하여, 더 깊이 있고 현실적인 이미지를 만듭니다.

11. **전역 조명 (Global Illumination, GI)**:
    - GI는 씬(scene) 내의 모든 물체들 사이에서의 빛의 상호작용을 계산하여, 보다 사실적인 조명을 구현합니다. 직접적인 빛뿐만 아니라 간접적인 빛(반사된 빛, 산란된 빛 등)도 고려합니다.
        1. **구면 조화(Spherical Harmonics, SH)**:
        - 저주파수의 조명 정보를 저장하여 간접 조명을 효율적으로 계산하는 방법입니다. 주로 실시간 렌더링에서 사용됩니다.<br/>
        [구면조화](/posts/0-구면조화(Spherical-Harnomics)/)

        2. **레이 트레이싱(Ray Tracing)**:
        - 카메라에서 시작한 빛의 경로를 추적하여 씬 내 모든 상호작용을 정확하게 계산하는 방법입니다. 거울 및 물 등의 반사에 매우 사실적이지만 계산 비용이 높습니다.

        3. **포톤 매핑(Photon Mapping)**:
        - 광원에서 시작한 빛의 경로를 따라 포톤을 발사하여 씬 내의 광원 상호작용을 시뮬레이션하는 방법입니다. 간접 조명과 반사를 효과적으로 처리합니다.<br/>
        이 정보를 텍스처에 저장한다면 유니티의 라이트맵핑과 유사하다고 보면 됩니다.

        4. **경로 추적(Path Tracing)**:
        - 광선 추적의 일종으로, 빛의 경로를 여러 번 샘플링하여 평균화하는 방법입니다. 현실적인 조명 효과를 구현하지만 계산 비용이 높습니다.

        5. **라디오시티(Radiosity)**:
        - 표면에서 반사된 빛이 다른 표면에 미치는 영향을 계산하여 씬의 조명을 시뮬레이션합니다. 주로 정적 씬에 사용됩니다.


## 다중 레이어 관련

12. **서브서페이스 스캐터링 (Subsurface Scattering, SSS)**:
    - 서브서페이스 스캐터링은 빛이 반투명 물질 내부로 들어가서 여러 번 산란되고 다시 표면으로 나오는 현상을 설명합니다.<br/>
    이는 피부, 왁스, 대리석 등에서 볼 수 있는 효과로, 물체의 깊이감을 나타내는 데 중요한 역할을 합니다.<br/>
    디퓨즈 근사,몬테카를로 시뮬레이션,데이터 기반, 가짜(두께맵을 사용한) 등이 있습니다.

13. **다중 레이어 재질 (Multi-Layer Materials)**:
     - PBR에서는 종종 여러 레이어로 구성된 재질을 사용하여 복잡한 표면 특성을 모델링합니다. 각 레이어는 다른 물리적 특성을 가지며, 이를 결합하여 보다 현실적인 재질을 구현합니다. 클리어코트, 피부, 나뭇잎, 꽃잎, 페인트, 금속코팅체 등이 존재하겠죠. 이전 비밀글이 였던, 원문 [Rendering Layered Materials with Diffuse Interfaces](https://arxiv.org/pdf/2203.11835)도 한번 읽어보세요.