---
layout: post
title: Dynamic Linq와 FakeNull
katex: True
categories: [Unity, Programming, Editor]
---
# 머리

다들 Unity의 FakeNull에 대해서는 많이 들어보셨을겁니다.

간략하게 initialize되지않은, destory 된 등 순수한 null은 아니지만 존재하지 않는 상태때문에

유니티에서는 ==연산자를 오버로딩했으며, 

```csharp

public static bool operator ==(Object x, Object y)
    => CompareBaseObjects(x, y);

private static bool CompareBaseObjects(Object lhs, Object rhs)
{
    bool flag = (object)lhs == null;
    bool flag2 = (object)rhs == null;
    if (flag2 && flag)
            return true;
    
    if (flag2)
        return !IsNativeObjectAlive(lhs);

    if (flag)
        return !IsNativeObjectAlive(rhs);

    return lhs.m_InstanceID == rhs.m_InstanceID;
}

```

아래와 같은 다른 다양한 방식들을 통해 이 FakeNull을 찾을 수 있도록 했습니다. 

```csharp
public static bool IsDestroyed(this UnityObject target)
    => !ReferenceEquals(target, null) && target == null;

public static bool IsUnityNull(this object obj)
    => obj == null || ((obj is UnityObject) && ((UnityObject)obj) == null);
```

# 몸통

하지만 문제가 있습니다.

Dynamic Linq에서 DynamicExpressionParser.ParseLambda를 통해 만들어진

LambdaExpression을 complie해서 사용하는 경우는 == 연산이 오버로딩된 ==(CompareBaseObjects)으로 처리하지 않고 그냥 ReferenceEquals와 같은방식으로 적용해버립니다…

a.font?.name 으로 하면 안되냐구요? 그거 dynamic linq에서는 np를 쓰는데 마찬가지입니다.

(애초에 null 조건부 연산자 자체가 이 상황에서 안먹습니다) 

쉽게 말하면 다음과 같습니다

```csharp
string.IsNullOrEmpty(a.font == null ? null : a.font.name)
np(a.font.Name) != null

//result : MissingReferenceException: The object of type 'Font' has been destroyed but you are still trying to access it.
```

아주 대중적인 fakeNull 발생시 볼 수 있는 상황입니다. 

그럼 문제가 생깁니다. 

기존 찾는 방식들은 다 오버로딩 된 ==와 object형의 ==의 차이를 통해 찾았는데… 

오버로딩된 ==(CompareBaseObjects) 를 사용할 수 없다면..?

그럼 Dynamic Linq 쿼리를 어떻게 짜야 FakeNull’만’ 찾도록 람다식을 구성할 수 있을까요?

답은 『ToString』입니다.

```csharp
a.font.ToString() == "null"
```

굉장히 간단하게도 fakeNull의 경우 ToString을 조지면 “null”이라는 4글자가 되어버리기 때문에 dynamicLinq에서도 컴파일한 람다식을 통해 페이크널을 검사할 수 있습니다. 

그러니 이제 이걸로 Missing 컴포넌트를 다 뒤질 수 있습니다.

# 꼬리

Dynamic Linq는 언제쓰냐고요?

저는 런타임에 무언가 의심될때 간단한 검색을 돌릴때 씁니다.

[https://gist.github.com/ashuatz/3221b37f78b7533fed9ea56b7b2431b9](https://gist.github.com/ashuatz/3221b37f78b7533fed9ea56b7b2431b9)