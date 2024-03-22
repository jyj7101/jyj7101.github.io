---
layout: post
title: 리플렉션 예제 - UnityEvent Args
katex: True
categories: [Unity, Programming, Editor]
---


# 머리

참조 할 수 없는 스크립트를 컴포넌트로 가진 게임오브젝트 인스턴스의 필드값,

심지어 그 필드값에 들어있는 실제 주요 직렬화 값은 internal한 type을 가진…

UnityEngine.Events.UnityEvent의 PersistentCall의 

ArgumentCache의 StringArgument 정도를 들고와서 비교….?

# 몸통

## 동적생성리스트

일단 만들어진 인스턴스는 동적으로 생성되었으므로, 코드작성단계에서 알 수 없음.
따라서 리플렉션으로 호출해야함. 이는 타입안전성이 없다-라고 봐야하므로 조심조심 사용.
그러므로 저는 무서우니 미리 구워서 처리.

기존 타입과, 만들어진리스트타입, 실제 객체를 들고있으며,

IList 형식이나 함수를 작성할 수 있도록 구성

```csharp

public class DynamicTypeList
{
    public readonly Type externalType;
    public readonly Type constructedListType;
    public readonly object listInstance;

    public IList list => listInstance as IList;

    private readonly Dictionary<string, MethodInfo> cachedMethoedInfos = new Dictionary<string, MethodInfo>();

    public DynamicTypeList(in Type T)
    {
        var listType = typeof(List<>);

        externalType = T;
        constructedListType = listType.MakeGenericType(new Type[] { externalType });

        listInstance = Activator.CreateInstance(constructedListType);
    }

    public DynamicTypeList(in Type T, in object value)
    {
        var listType = typeof(List<>);

        externalType = T;
        constructedListType = listType.MakeGenericType(new Type[] { externalType });

        listInstance = value;
    }

    //method call example
    public void Add(in object value)
    {
        if(value.GetType() != externalType)
            throw new TypeAccessException();

        if (!cachedMethoedInfos.TryGetValue(nameof(Add), out var info))
        {
            info = constructedListType.GetMethod(nameof(Add));
            cachedMethoedInfos[nameof(Add)] = info;
        }

        info.Invoke(listInstance, new object[] { value });
    }

}
```

## 실제구현

두 부분으로 나눌 수 있습니다.

유니티이벤트를 가져오는 부분, 유니티이벤트에서 문자열을 가져오는 부분

```csharp
//Get UnityEvent-Field via reflection
private bool PredictObject(in GameObject target)
{
    var comps = target.GetComponents<Component>();
    foreach (var component in comps)
    {
        var fields = component.GetType().GetFields(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance);

        foreach (var field in fields)
        {
            if (field.FieldType == typeof(UnityEngine.Events.UnityEvent))
            {
                var value = field.GetValue(component);
                GetUnityEventStringArgs(value as UnityEngine.Events.UnityEvent, out var args);
                
                if (args.Any(str => str.Equals("<Target Arg>")))
		                return true;
            }
        }
    }
    return false;
}
```

유니티이벤트에서 문자열을 가져오는 부분

```csharp
//Get UnityEvent's string args
private void GetUnityEventStringArgs(in UnityEngine.Events.UnityEvent e, out List<string> stringArgs)
{
    //depth 1. UnityEngine.CoreModule - UnityEngine.Events
    Assembly CoreModuleAssembly = Assembly.Load("UnityEngine.CoreModule");
    Type PersistentCallType = CoreModuleAssembly.GetType("UnityEngine.Events.PersistentCall");
    //depth 2. UnityEventBase's PersistentCallGroup
    var f_group = e.GetType().BaseType.GetField("m_PersistentCalls", BindingFlags.NonPublic | BindingFlags.Instance);
    var group = f_group.GetValue(e);
    
    //depth 3. PersistentCallGroup's List<PersistentCall> m_Calls
    var f_calls = group.GetType().GetField("m_Calls", BindingFlags.NonPublic | BindingFlags.Instance);
    var calls = new DynamicTypeList(PersistentCallType, f_calls.GetValue(group));

    stringArgs = new List<string>();
    foreach (var call in calls.list)
    {
        //depth 4. PersistentCall's ArgumentCache m_Arguments
        var f_cache = call.GetType().GetField("m_Arguments", BindingFlags.NonPublic | BindingFlags.Instance);
        var cache = f_cache.GetValue(call);

        //depth 5. ArgumentCache's string m_StringArgument
        var f_stringArgs = cache.GetType().GetField("m_StringArgument", BindingFlags.NonPublic | BindingFlags.Instance);
        var strArg = f_stringArgs.GetValue(cache);
        stringArgs.Add(strArg as string);
    }
}
```

# 꼬리

PersistentCalls랑 InvokableCallList랑 다르긴한데

런타임에 동적으로 UnityEvent에 AddListener한건 InvokableCallList에 , 

직렬화하여 넣은 ‘인스펙터에서 확인되는 부분’은 PersistentCalls에 들어갑니다.

혹은 UnityEventTools.AddPersistentListener 로 추가해도 PersistentCalls에 들어갑니다.

그리고 리플렉션 느리니까 사용용도와 목적에 맞게 사용합시다.

저는 자동화코드에 사용함.

![Q.E.D](/assets/Reflection_Unity/Untitle.gif)
*Q.E.D*