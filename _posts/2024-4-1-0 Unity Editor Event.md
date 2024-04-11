---
layout: post
title: 가끔쓰는 유니티 에디터 이벤트 메모
katex: True
categories: [Unity, Programming, Editor]
---

매번 찾기 귀찮아서 메모

# 에디터 종료시 확인버튼

wantsToQuit  → func<bool> false 반환시 안꺼짐

quitting → Action 

```csharp
EditorApplication.wantsToQuit += ()=> someCondition;
```

# 라이트맵

구울때 async로 굽고 대충 웹훅으로 메시지 보내게 하면 됩니다 굳?굳.

```csharp
Lightmapping.bakeStarted += () => { };
Lightmapping.bakeCompleted += () => { };
```

# 다이얼로그

크리티컬한 에디터작업 전 확인메시지. Undo도 쓰면 굳

```csharp
var confirmed = EditorUtility.DisplayDialog("Title","message","ok","cancel");
```

# 빌드

인터페이스. 빌드전, 빌드후 나눌 수 있으며 인터페이스 구현필요.

코드는 귀찮으니 유니티 레퍼문서

```csharp
class MyCustomBuildProcessor : IPostprocessBuildWithReport
{
    public int callbackOrder { get { return 0; } }
    public void OnPostprocessBuild(BuildReport report)
    {
        Debug.Log("MyCustomBuildProcessor.OnPostprocessBuild for target " + report.summary.platform + " at path " + report.summary.outputPath);
    }
}

class MyCustomBuildProcessor : IPreprocessBuildWithReport
{
    public int callbackOrder { get { return 0; } }
    public void OnPreprocessBuild(BuildReport report)
    {
        Debug.Log("MyCustomBuildProcessor.OnPreprocessBuild for target " + report.summary.platform + " at path " + report.summary.outputPath);
    }
}
```