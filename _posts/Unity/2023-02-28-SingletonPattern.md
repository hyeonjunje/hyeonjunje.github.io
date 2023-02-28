---
layout: single
title:  "[Unity] Unity에서 사용되는 싱글톤 패턴"
categories: Unity
tag: [Unity, Design Pattern]
# search: false 검색 기능에 나오지 않게 할려면 false
---

싱글톤은 기본 디자인 패턴입니다. <br>
싱글톤 패턴을 구현하는 클래스는 그 객체의 인스턴스가 하나만 존재하도록 합니다. <br>
게임에서는 GameManager나 AudioController, EffectManager 등등 단 하나의 오브젝트만 필요한 경우가 있습니다. <br>
그 편이 더 관리하기도 쉽고 외부에서 가져오는 것도 간편하죠. <br>

오늘은 유니티에서 사용할 수 있는 싱글톤 패턴에 대해서 포스팅 하겠습니다. <br>
해당 내용은 [http://www.unitygeek.com/unity_c_singleton/](http://www.unitygeek.com/unity_c_singleton/)을 참조하였습니다. <br><br>

# 구현 방법 1 : 가장 간단한 싱글톤

``` c# 
public class SingletonController : MonoBehaviour
{
    public static SingletonController instance;

    private void Awake()
    {
        if(instance == null)
            instance = this;
    }
}
```

가장 짧고 간단한 구현방법입니다. <br>
저도 이 방법을 많이 사용했는데 이 방법은 간단한 만큼 허술한 점이 많습니다. <br>
문제점
1. 유니티 모든 Scene에 지속되지 않습니다.
2. 유니티의 Hierarchy에 GameObject를 생성하고 할당해야 합니다.
3. instance가 Awake문에서 결정됩니다. 모든 스크립트의 실행 순서는 랜덤이기에 SingletonController의 Awake 전에 
instance를 호출한다면 Null 참조 예외가 발생합니다.
4. SingletonController 클래스에만 작용합니다. 예를 들어 AudioController를 싱글톤으로 만들기 위해선 위의 코드를 
똑같이 써야 한다는 불편함이 있습니다.

그럼 이 문제점들을 고쳐봅시다. <br>
<br>

# 구현 방법 2 : 모든 씬에 지속되는 싱글톤 (1번 문제 해결)
``` c# 
public class SingletonController : MonoBehaviour
{
    public static SingletonController instance;

    private void Awake()
    {
        if(instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameobject);
        }
        else
            Destroy(gameobject);
    }
}
```
모든 씬에 지속되기를 원한다면 단순히 DontDestroyOnLoad를 사용하면 됩니다. <br>
그리고 싱글톤인 오브젝트가 아닌 오브젝트가 생긴다면 파괴해주면 됩니다. <br>
<br>

# 구현 방법 3 : 필요한 시점에 싱글톤 생성 (2번, 3번 문제 해결)
``` c#
public class SingletonController : MonoBehaviour
{
    private static SingletonController instance = null;
    public static SingletonController Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindObjectOfType<SimpleSingleton>();
                if (instance == null)
                {
                    GameObject go = new GameObject();
                    go.name = "SingletonController";
                    instance = go.AddComponent<SingletonController>();
                    DontDestroyOnLoad(go);
                }
            }
            return instance;
        }
    }

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(this.gameObject);
        }
        else
            Destroy(gameObject);
    }
}
```

Awake에서도 instance를 할당하지만 getter, setter를 활용해 Instance가 불릴 때 아직 instance가 null이라면
게임오브젝트를 생성하고 해당 컴포넌트를 넣어줍니다. <br>
이러면 하이어라키 창에 항상 GameObject를 만들고 해당 컴포넌트를 넣어야 하는 번거로움이 없습니다. <br>
그리고 Instance로 다른 곳에서 불릴 때 해당 오브젝트가 생성되기 때문에 스크립트 실행 순서와는 상관이 없어졌습니다. <br>
<br>

# 구현 방법 4 : 재사용성을 높인 코드 (4번 문제 해결)
``` c#
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    public static T Instance
    {
        get
        {
            if(instance == null)
            {
                instance = FindObjectOfType<T>();
                if(instance == null)
                {
                    GameObject obj = new GameObject();
                    obj.name = typeof(T).Name;
                    instance = obj.AddComponent<T>();
                }
            }
            return instance;
        }
    }

    public virtual void Awake()
    {
        if(instance == null)
        {
            instance = this as T;
            DontDestroyOnLoad(this.gameObject);
        }
        else
            Destroy(gameObject);
    }
}
```

abstract로 추상클래스로 만들고 Generic으로 MonoBehaviour를 상속받는 클래스를 싱글톤으로 만들 수 있습니다.
코드를 복제하지 않고 단순히 상속을 받으면 되죠.

``` c#
public class GameManager : Singleton<GameManager>
{
   public override void Awake()
   {
    base.Awake();
   } 
}
```

하지만 여기서도 문제가 있습니다. 
1. 싱글톤을 상속받을 때 Awake를 override한다는게 번거롭습니다.
2. Instance를 호출할 때 null이라면 싱글톤을 생성하는데 만약 프로그램이 종료됐을 때 해당 싱글톤이 파괴가 되고
싱글톤을 호출할 경우 또 객체가 생성될 수 있습니다.

이러한 문제를 해결해봅시다. <br><br>


# 언급한 문제를 모두 해결한 싱글톤 코드
``` c#
// 오브젝트에 해당 컴포넌트를 복수개 붙일 수 없게 하는 어트리뷰트
[DisallowMultipleComponent]
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;
    private static bool isApplicationQuit = false;

    public static T Instance
    {
        get 
        {
            if (isApplicationQuit == true)
                return null;

            if(instance == null)
            {
                T[] _finds = FindObjectsOfType<T>();
                if(_finds.Length > 0)
                {
                    instance = _finds[0];
                    DontDestroyOnLoad(instance.gameObject);
                }

                if(_finds.Length > 1)
                {
                    for(int i = 1; i < _finds.Length; i++)
                        Destroy(_finds[i].gameObject);
                }

                if(instance == null)
                {
                    GameObject _createGameObject = new GameObject(typeof(T).Name);
                    DontDestroyOnLoad(_createGameObject);
                    instance = _createGameObject.AddComponent<T>();
                }
            }
            return instance;
        }
    }

    private void OnApplicationQuit()
    {
        isApplicationQuit = true;
    }
}
```

Awake에서 하던 코드를 전부 get 프로퍼티에 넣었습니다. 따라서 Awake를 override 할 필요가 없어졌죠. <br>
그리고 하이어라키에 해당 클래스가 붙은 오브젝트가 다수개 있을 때 제일 처음 오브젝트 빼고 다 Destroy처리해줬습니다. <br>
또 프로그램이 종료될 때 Instance를 null로 반환하도록 하여 다시 생성하는 것을 막았습니다. <br>

마지막으로 어트리뷰트를 이용해 한 오브젝트에 싱글톤 컴포넌트를 복수개 붙일 수 없게 하는 DisallowMultipleComponent를 추가했습니다. <br>

그럼 싱글톤을 사용하고 싶은 곳이 있다면 어떤 구현없이 단순히 해당 클래스를 상속받으면 됩니다. <br>
매우 객체지향적이군요!! <br>