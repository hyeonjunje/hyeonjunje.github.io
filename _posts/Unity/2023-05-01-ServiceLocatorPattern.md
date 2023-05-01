---
layout: single
title:  "[Unity] Unity에서의 ServiceLocator패턴"
categories: Unity
tag: [Unity, Design Pattern]
# search: false 검색 기능에 나오지 않게 할려면 false
---

이번에 Unity로 "Mega Crit Games"에서 개발한 "Slay The Spire"모작하면서 <br>
많은 manager를 관리해야할 일이 있습니다. <br>

모든 manager를 static으로 하면 쓸데없이 메모리를 많이 잡아먹을 수 있고, <br>
singleton으로 다 구현하면 관리하거나 사용하는 것이 복잡합니다. <br>
또한 직접적인 의존으로 인해 코드간 결합도가 높아져 유지보수에도 문제가 생길 수 있습니다. <br>

그래서 ServiceLocator라는 패턴을 사용해보기로 했습니다!! <br><br>

# ServiceLocator 패턴
<br>
ServiceLocator 패턴은 소프트웨어 디자인 패턴 중 하나로, 서비스 객체를 생성하고 관리하는데 사용합니다. <br>
이 패턴은 각 서비스 객체에 대한 의존성을 갖지 않으며, 이들 서비스를 사용할 수 있는 중앙 집중화된 위치를 제공합니다. <br>


ServiceLocator 패턴은 다음과 같은 구성요소로 이루어집니다. <br>
> Service : 인터페이스로 정의된 서비스 객체입니다. <br>
> Service Implementation : Service 인터페이스를 구현한 서비스 객체입니다. <br>
> Service Locator : 서비스 객체의 인스턴스를 찾고 반환하는데 사용되는 클래스입니다. <br>
> Client : ServiceLocator를 사용하여 필요한 서비스 객체를 얻는 클라이언트 코드입니다. <br>


보통 ServiceLocator패턴은 Singleton 패턴과 함께 사용됩니다. <br>
이를 통해 ServiceLocator 인스턴스를 하나만 만들어 전체 애플리케이션에서 공유할 수 있으며, <br>
각 서비스 객체는 ServiceLocator를 통해 참조됩니다. <br>


ServiceLocator 패턴은 서비스 객체를 중앙 집중화하는 데 사용됩니다. 이를 통해 서비스 객체의 <br>
생성과 구성을 단순화하고, 코드의 유연성과 재사용성을 높일 수 있습니다. <br>
그러나 이 패턴은 ServiceLocator 자체에 대한 의존성이 생길 수 있으며, <br>
이를 사용하는 코드가 복잡해질 수 있습니다. <br>

<br><br>

# 유니티에서 활용



``` c#
public interface IRegisterable
{
    public void Init();
}

public class ServiceLocator : MonoBehaviour
{
    public static ServiceLocator instance;
    public static ServiceLocator Instance
    {
        get
        {
            if(instance == null)
            {
                instance = FindObjectOfType<ServiceLocator>();
                instance.Init();
            }
            return instance;
        }
    }

    private IDictionary<object, IRegisterable> services;

    private void Init()
    {
        services = new Dictionary<object, IRegisterable>();
    }

    public T GetService<T>() where T : MonoBehaviour, IRegisterable
    {
        if (!services.ContainsKey(typeof(T)))
        {
            T manager = FindObjectOfType<T>();
            if(manager != null)
            {
                services[typeof(T)] = manager;
                manager.Init();

                return (T)services[typeof(T)];
            }

            Debug.LogError("ServiceLocator::GetService 없는 키입니다.");
            return default(T);
        }

        return (T)services[typeof(T)];
    }
}
```

우선 서비스로 등록할 메니저들은 IRegisterable 인터페이스를 상속받도록 합니다. <br>
ServiceLocator 클래스는 Singleton으로 만들어줍니다. <br>
Singleton은 간단하게 만들어줬습니다. <br>

클라이언트에서 가져올 메니저(서비스)들을 가져올 때는 GetService 메소드로 가져옵니다. <br>
Monobehaviour과 IRegisterable를 상속받는 제네릭만 가져올 수 있게 합니다. <br>

GetService<GameManager>(), GetService<SoundManager> 등등으로 가져올 수 있습니다. <br>
메니저들을 가져올 때는 services dictionary에 있으면 그대로 가져오고 <br>
없다면 찾아서 등록해주고 가져옵니다. <br>
여기서 초기화 작업을 해줘야 한다면 IRegisterable 인터페이스에 Init()함수를 강제로 만들게 하여 실행시킬 수도 있습니다.<br>






