---
layout: post
title: Kuwahara filter
katex: True
categories: [Graphic, Shader, Unity]
---

쿠와하라(Kuwahara) 필터는 노이즈제거 및 엣지 보존 기술입니다.

# **원리**

1. **이미지 분할**: 이미지를 여러 개의 작은 영역으로 나눕니다. 일반적으로 각 픽셀을 중심으로 하는 5x5 크기의 윈도우가 사용됩니다. 여기서는 (_KernelSize * 2 + 1) * 2 의 윈도우를 사용합니다.
2. **지역 통계 계산**: 각 윈도우를 네 개의 부분으로 나누고, 각 부분에서 평균과 분산을 계산합니다.
3. **최소 분산 선택**: 각 픽셀에 대해, 네 부분 중 분산이 가장 작은 부분의 평균값으로 픽셀 값을 대체합니다. 이는 노이즈가 적고 텍스처가 일관된 영역을 선택하는 효과를 줍니다.

포스트프로세싱이 다 그렇듯 결국 주변픽셀과의 연산입니다.

간단하게 커널사이즈와 패스숫자로만 조절하겠습니다.

# 결과물

적당한 타협을 본다면 런타임중에 사용할 수 있습니다.

런타임중 패스와 커널사이즈를 수정할 수 있습니다.

물속에 잠수한다던지, 조금 다른 시각적 감각을 보여주고 싶을 때, 사용하면 괜찮을 것 같습니다.

![sample_Kuwahara_2.gif](/assets/kuwahara/sample_Kuwahara_2.gif)
*교내 졸업작품 LUKA : IN THE AZURE FIELD에 임시로 작업해본 샘플입니다*

![GIF1.gif](/assets/kuwahara/GIF1.gif)

파라메터를 조정하며 저번에 사용한 삼각분할반사버전 오브젝트도 넣어봤습니다.

![GIF2.gif](/assets/kuwahara/GIF2.gif)

# 구현

그러니 간단하게 렌더피처부터 구현

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class KuwaharaRenderFeature : ScriptableRendererFeature
{
    [Serializable]
    public class KuwaharaProperty
    {
        [Range(1, 20)]
        public int kernelSize = 1;

        [Range(1, 4)]
        public int passes = 1;
    }

    [System.Serializable]
    public class KuwaharaSettings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.AfterRenderingTransparents;
        public Material passMaterial = null;

        public KuwaharaProperty property;
    }

    public KuwaharaSettings settings = new KuwaharaSettings();

    class KuwaharaRenderPass : ScriptableRenderPass
    {
        //property
        public KuwaharaProperty property;

        public Material kuwaharaMat;
        string profilerTag;

        RenderTargetIdentifier cameraColorTexture;

        public KuwaharaRenderPass(string profilerTag)
        {
            this.profilerTag = profilerTag;
        }

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
        {
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            cameraColorTexture = renderingData.cameraData.renderer.cameraColorTarget;
            CommandBuffer cmd = CommandBufferPool.Get(profilerTag);

            RenderTextureDescriptor opaqueDesc = renderingData.cameraData.cameraTargetDescriptor;
            opaqueDesc.depthBufferBits = 0;

            //
            Process();

            //
            context.ExecuteCommandBuffer(cmd);
            cmd.Clear();

            CommandBufferPool.Release(cmd);

            void Process()
            {
                kuwaharaMat.SetInt("_KernelSize", property.kernelSize);

                var passes = new RenderTexture[property.passes];

                for (int i = 0; i < property.passes; ++i)
                {
                    passes[i] = RenderTexture.GetTemporary(opaqueDesc.width, opaqueDesc.height, 0, opaqueDesc.colorFormat);
                }

                cmd.Blit(cameraColorTexture, passes[0], kuwaharaMat);

                for (int i = 1; i < property.passes; ++i)
                {
                    cmd.Blit(passes[i - 1], passes[i], kuwaharaMat);
                }

                cmd.Blit(passes[property.passes - 1], cameraColorTexture);

                //release
                for (int i = 0; i < property.passes; ++i)
                {
                    RenderTexture.ReleaseTemporary(passes[i]);
                }
            }

        }

        public override void FrameCleanup(CommandBuffer cmd)
        {

        }
    } // end KuwaharaRenderPass

    private KuwaharaRenderPass scriptablePass;

    public override void Create()
    {
        scriptablePass = new KuwaharaRenderPass("kuwahara");
        scriptablePass.kuwaharaMat = settings.passMaterial;
        scriptablePass.renderPassEvent = settings.renderPassEvent;
        scriptablePass.property = settings.property;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        renderer.EnqueuePass(scriptablePass);
    }
}

```

## 쉐이더 구현

```hlsl
Shader "Hidden/KuwaharaShader"
{
    Properties
    { 
        [HideInInspector]_MainTex ("Texture", 2D) = "white" {}
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalRenderPipeline" }

        Cull Off ZWrite Off ZTest Always
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"            

            struct Attributes
            {
                float4 positionOS   : POSITION;      
                float2 uv : TEXCOORD0;           
            };

            struct Varyings
            {
                float2 uv : TEXCOORD0;
                float4 positionHCS  : SV_POSITION;
            };            

            CBUFFER_START(UnityPerMaterial)
                sampler2D _MainTex;                    
                uniform half4 _MainTex_TexelSize;

                int _KernelSize;
            CBUFFER_END

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = IN.uv;
                return OUT;

            }

            float luminance(float3 color)
            {
                return dot(color, float3(0.299f, 0.587f, 0.114f));
            }
            
            float4 SampleQuadrant(float2 uv, int x1, int x2, int y1, int y2, float n)
            {
                float luminance_sum = 0.0f;
                float luminance_sum2 = 0.0f;
                float3 col_sum = 0.0f;

                [loop]
                for (int x = x1; x <= x2; ++x)
                {
                    [loop]
                    for (int y = y1; y <= y2; ++y)
                    {
                        float3 sample = tex2D(_MainTex, uv + float2(x, y) * _MainTex_TexelSize.xy).rgb;
                        float l = luminance(sample);
                        luminance_sum += l;
                        luminance_sum2 += l * l;
                        col_sum += saturate(sample);
                    }
                }

                float mean = luminance_sum / n;
                float std = abs(luminance_sum2 / n - mean * mean);

                return float4(col_sum / n, std);
            }

            half4 frag(Varyings i) : SV_Target
            {
                float windowSize = 2.0f * _KernelSize + 1;
                int quadrantSize = int(ceil(windowSize / 2.0f));
                int numSamples = quadrantSize * quadrantSize;

                float4 q1 = SampleQuadrant(i.uv, -_KernelSize, 0, -_KernelSize, 0, numSamples);
                float4 q2 = SampleQuadrant(i.uv, 0, _KernelSize, -_KernelSize, 0, numSamples);
                float4 q3 = SampleQuadrant(i.uv, 0, _KernelSize, 0, _KernelSize, numSamples);
                float4 q4 = SampleQuadrant(i.uv, -_KernelSize, 0, 0, _KernelSize, numSamples);

                float minstd = min(q1.a, min(q2.a, min(q3.a, q4.a)));
                int4 q = float4(q1.a, q2.a, q3.a, q4.a) == minstd;
                
                if (dot(q, 1) > 1)
                    return saturate(float4((q1.rgb + q2.rgb + q3.rgb + q4.rgb) / 4.0f, 1.0f));
                else
                    return saturate(float4(q1.rgb * q.x + q2.rgb * q.y + q3.rgb * q.z + q4.rgb * q.w, 1.0f));
            }
            ENDHLSL
        }
    }
}

```