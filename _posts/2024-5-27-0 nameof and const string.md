---
layout: post
title: nameof와 const string
katex: True
categories: [Programming]
---

요약 : 런타임 성능 똑같음

(이번엔 간단히.. 수술로 1주일이 날라갔습니다)

# 머리

```csharp
void GetData(in DerivedType instance)
{
    FieldInfo fieldInfo = typeof(BaseType).GetField(nameof(m_targetData), BindingFlags.NonPublic | BindingFlags.Instance);
    m_targetData = fieldInfo.GetValue(instance) as TargetType;
}
```

리플렉션을 사용하다보면 위같은 코드를 작성할 때가 있습니다.

그러면 궁금한점이 있습니다.

리플렉션을 사용해서 멤버를 가져올때, 멤버의 이름이 “m_targetData”라면 

const string fieldName = “m_targetData” 을 사용하는게 나을까요,

아니면 가져올 데이터도 동일한 이름을 사용해서, nameof로 작성하는게 나을까요?

# 몸통

## 성능

<code>nameof</code>와 비교했을 때 <code>const string</code> 은 성능 차이는 거의 없습니다. 

둘 다 컴파일 시간에 문자열을 결정하므로, 런타임 성능에는 영향을 미치지 않습니다.

![Untitled](assets/nameof_const_string/Untitled.png)

3개의 대조군을 IL과 JIT 어셈블리로 확인해본다면,

![Untitled](assets/nameof_const_string/Untitled%201.png)

IL 동일,

![Untitled](assets/nameof_const_string/Untitled%202.png)

어셈블리에서도 동일한 주소를 지정함을 알 수 있습니다.

## 유지보수 안전성

<code>nameof</code>는 보통 변수나 속성의 이름이 변경될 때 자동으로 업데이트되므로
(Rename : vs - ctrl R R, vscode - f2) , 리팩토링 시 오류를 줄일 수 있습니다.

다만 ‘리플렉션을 사용하는 부분’에서는 이 부분이 치명적인 단점으로 동작할 수 있습니다. 

베이스타입의 멤버변수의 이름이 바뀌는 경우에는 <code>nameof</code>나 <code>const string</code>이나 둘 다 수정해야하니 어쩔 수 없지만, ‘리플렉션을 사용하는 코드’에서 다른 작업자가 컨벤션에 맞춰 수정한다-라는 가정이 들어가면 예상치 못한 휴먼에러가 발생할 수 있습니다.

# 꼬리

![이런 결말에 어울리는 이모티콘](assets/nameof_const_string/Untitled%203.png)\
*이런 결말에 어울리는 이모티콘*

GPT식 엔딩으로, 따라서 어떤 방법이 더 나은지는 사용 사례에 따라 달라집니다.

+

![Untitled](assets/nameof_const_string/Untitled%204.png)

그리고 애초에 리플렉션의 저 기능들. Get ~ MethodInfo, PropertyInfo, FieldInfo, CustomAttributes 그리고 Invoke류는 비쌉(costly)니다