# 데코레이터 패턴

```cpp
struct Shape
{
    virtual std::string str() const = 0;
};

struct Circle : Shape
{
    float radius;

    explicit Circle(const float radius) :
        radius{ radius }
    {}

    void resize(float factor)
    {
        radius *= factor;
    }

    std::string str() const override
    {
        std::ostringstream oss;
        oss << "A circle of radius " << radius;
       ret urn oss.str(); 
    }
};
```

## 동적 컴포지션

* 구조 예시

```cpp
struct ColoredShape : Shape
{
    Shape & shape;
    std::string color;

    ColoredShape(Shape & shape, const std::string & color) ;
        shape { shape },
        color { color }
    {}

    std::string str() const override
    {
        std::ostringstream oss;
        oss << shape.str() << " has the color " << color;
        return oss.str();
    }
};

struct TransparentShape : Shape
{
    Shape & shape;
    uint8_t transparency;

    TransparentShape(Shape & shape, const uint8_t transparency) ;
        shape { shape },
        transparency { transparency }
    {}

    std::string str() const override
    {
        std::ostringstream oss;
        oss << shape.str() << " has " <<
        static_cast<float>(transparency) / 255.f * 100.f << "% transparency";
        return oss.str();
    }
};
```

* 사용 예시

```cpp
// correct usage

Circle circle{ 0.5f };
ColoredShape redCircle{circle, "red"};
std::cout << redCircle.str();

Square square{ 3 };
TransparentShape demiSquare{square, 85};
std::cout << demiSquare.str();

TransparentShape myCircle{
    ColoredShape{
        Circle{23}, "green"
    }, 64
};
std::cout << myCircle.str();

// wrong usage

ColoredShape{ColoredShpae{...}}

ColoredShape{TransparentShape{ColoredShape{...}}}
```

## 정적 컴포지션

* 구조 예시

```cpp
template<class T>
struct ColoredShape : T
{
    static_assert(std::is_base_of_v<Shape, T>,
    "Template argument must be a Shape");

    std::string color;

    template<class... Args>
    ColoredShape(const std::string & color, Args... args) :
        T{std::forward<Args>(args)...},
        color{ color }
    {}

    std::string str() const override
    {
        std::ostringstream oss;
        oss << T::str() << " has the color " << color;
        return oss.str();
    }
};
// ...
```

* 사용 예시

```cpp
ColoredShape<TransparentShape<Square>> sq = {"red", 51, 5};
std::cout << sq.str();
```

## 함수형 데코레이터

* 구조 예시

```cpp
struct Logger
{
    std::function<void()> func;
    std::string name;

    Logger(const std::function<void()> & func, const std::string & name) :
        func{ func },
        name{ name }
    {}

    void operator() () const
    {
        std::cout << "Entering " << name << "\n";
        func();
        std::cout << "Exiting " << name << "\n";
    }
};

template<class Func>
struct Logger2
{
    Func func;
    std::string name;

    Logger(const Func & func, const std::string & name) :
        func{ func },
        name{ name }
    {}

    void operator() () const
    {
        std::cout << "Entering " << name << "\n";
        func();
        std::cout << "Exiting " << name << "\n";
    }
};

template<class Func>
auto make_logger2(Func & func, const string & name)
{
    return Logger2<Func>{func, name};
}

template<class R, class... Args>
struct Logger3
{
    std::function<R(Args...)> func;
    std::string name;

    Logger(const std::function<R(Args...)> & func, const std::string & name) :
        func{ func },
        name{ name }
    {}

    R operator() (Args... args) const
    {
        std::cout << "Entering " << name << "\n";
        R result = func(args...);
        std::cout << "Exiting " << name << "\n";
    }
};

template<class R, class... Args>
auto make_loggerr(R(*func)(Args...), const string & name)
{
    return Logger3<R(Args...)>{std::function<R(Args...)>{func}, name};
}
```
