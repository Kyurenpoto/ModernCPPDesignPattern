# 모던 C++로 배우는 디자인 패턴

## 중요 개념

### CRTP

```cpp
struct Foo : SomeBase<Foo>
{
    ...
};
```

* 베이스 클래스의 구현부에서 this를 하위 타입으로 캐스팅 가능

### Mixin 상속

```cpp
teamplate<class T> struct Mixin : T
```

* 계층적으로 여러 타입을 합성

## SOLID

### SRP(단일 책임 원칙)

* 각 클래스는 단 하나의 책임을 부여받아 수정을 할 이유 또한 단 하나여야 함

### OCP(열림-닫힘 원칙)

* 기존 코드의 수정 없이 코드를 확장
* 인터페이스에 손을 대지 않고, 그 구현을 통해 기능을 확장 가능하게 만들어져야 함

### LSP(리스코프 치환 원칙)

* 자식 객체에 접근할 때, 그 부모의 인터페이스로 접근해도 문제가 없어야 함

### ISP(인터페이스 분리 원칙)

* 필요에 따라 구현할 대상을 선별할 수 있도록 인터페이스를 별개로 두어야 함

### DIP(의존성 역전 원칙)

* 추상화는 세부사항에 의존해서는 안되며, 그 역이 되어야 함
* 의존성 주입 예시

```cpp
struct Engine
{
    float volume = 5;
    int horse_power = 400;

    friend std::ostream & operator << (std::ostream & os, const Engine & obj)
    {
        return os <<
               "volume: " << obj.volume <<
               "horse_power: " << obj.horse_power;
    }
};

struct ILogger
{
    virtual ~ILogger() = default;
    virtual void Log(const std::string & s) = 0;
};

struct ConsoleLogger : ILogger
{
    ConsoleLogger() = default;

    void Log(const std::string & s) override
    {
        cout << "LOG: " << s.c_str() << endl;
    }
};

struct Car
{
    std::unique_ptr<Engine> engine;
    std::shared_ptr<ILogger> logger;

    Car(std::unique_ptr<Engine> engine,
        const std::shared_ptr<ILogger> & logger) :
        engine{ std::move(engine) },
        logger{ logger }
    {
        logger->Log("making a car");
    }

    friend std::ostream & operator << (std::ostream & os, const Car & obj)
    {
        return os << "car with engine: " << *obj.engine;
    }
};

auto injector = di::make_injector(di::bind<ILogger>().to<ConsoleLogger>());
auto car = injector.create<std::shared_ptr<Car>>();
```

## 생성 패턴

* [빌더 패턴](creation/builder.md)
* [팩토리 패턴](creation/factory.md)
* [프로토타입 패턴](creation/prototype.md)
* [싱글턴 패턴](creation/singleton.md)

## 구조 패턴

* [어댑터 패턴](structure/adapter.md)
* [브릿지 패턴](structure/bridge.md)
* [컴포지트 패턴](structure/composite.md)
* [데코레이터 패턴](structure/decorator.md)
* [퍼사드 패턴](structure/facade.md)
* [플라이웨이트 패턴](structure/flyweight.md)
* [프록시 패턴](structure/proxy.md)

## 행동 패턴

* [책임 사슬 패턴](behavior/chain_responsibility.md)
* [커맨드 패턴](behavior/command.md)
* [인터프리터 패턴](behavior/interpreter.md)
* [반복자 패턴](behavior/iterator.md)
* [매개자 패턴](behavior/mediator.md)
* [메멘토 패턴](behavior/memento.md)
* [Null 객체 패턴](behavior/null_object.md)
* [관찰자 패턴](behavior/observer.md)
* [상태 패턴](behavior/state.md)
* [전략 패턴](behavior/strategy.md)
* [템플릿 메서드 패턴](behavior/template_method.md)
* [방문자 패턴](behavior/visitor.md)

## 함수형 패턴

* [Maybe 모나드 패턴](functional/maybe_monad.md)
