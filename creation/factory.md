# 팩토리 패턴

## 팩토리 메서드

* 구조 예시

```cpp
struct Point
{
protected:
    Point(const float x, const float y) :
        x{ x },
        y{ y }
    {}

public:
    static Point NewCartesian(float x, float y)
    {
        return { x, y };
    }

    static Point NewPolar(float r, float theta)
    {
        return { r * cos(theta), r * sin(theta) };
    }

    float x, y;
};
```

* 사용 예시

```cpp
auto p = Point::NewPolar(5, M_PI_4);
```

## 팩토리

* 구조 예시

```cpp
struct PointFactory;

struct Point
{
    float x, y;
    friend class PointFactory;

private:
    Point(const float x, const float y) :
        x{ x },
        y{ y }
    {}
};

struct PointFactory
{
    static Point NewCartesian(float x, float y)
    {
        return Point{ x, y };
    }

    static Point NewPolar(float r, float theta)
    {
        return Point{ r * cos(theta), r * sin(theta) };
    }
};
```

* 사용 예시

```cpp
auto p = PointFactory::NewCartesian(3, 4);
```

## 내부 팩토리

* 구조 예시

```cpp
struct Point
{
    float x, y;

    struct Factory
    {
        static Point NewCartesian(float x, float y)
        {
            return { x, y };
        }

        static Point NewPolar(float r, float theta)
        {
            return { r * cos(theta), r * sin(theta) };
        }
    };

private:
    Point(const float x, const float y) :
        x{ x },
        y{ y }
    {}
};
```

* 사용 예시

```cpp
auto pp = Point::Factory::NewCartesian(3, 4);
```

## 추상 팩토리

* 구조 예시

```cpp
struct HotDrink
{
    virtual void prepare(int volume) = 0;
};

struct Tea : HotDrink
{
    void prepare(int volume) override
    {
        std::cout << "Take tea bag, boil water pour " << volume <<
            "ml, add some lemon\n";
    }
};

struct Coffee : HotDrink
{
    void prepare(int volume) override
    {
        std::cout << "Take coffee bean, boil water pour " << volume <<
            "ml, add sugar\n";
    }
};

struct HotDrinkFactory
{
    virtual std::unique_ptr<HotDrink> make() const = 0;
};

struct TeaFactory : HotDrinkFactory
{
    std::unique_ptr<HotDrink> make() const override
    {
        return make_unique<Tea>();
    }
};

struct CoffeeFactory : HotDrinkFactory
{
    std::unique_ptr<HotDrink> make() const override
    {
        return make_unique<Coffee>();
    }
};

struct DrinkFactory
{
    DrinkFactory()
    {
        hot_factories["tea"] = make_unique<TeaFactory>();
        hot_factories["coffee"] = make_unique<CoffeeFactory>();
    }

    std::unique_ptr<HotDrink> make_drink(const std::string & name)
    {
        auto drink = hot_factories[name]->make();
        drink->prepare(200);
        return drink;
    }

private:
    std::map<std::string, std::unique_ptr<HotDrinkFactory>> hot_factories;
};
```

## 함수형 팩토리

* 구조 예시

```cpp
struct DrinkWithVolumeFactory
{
    DrinkWithVolumeFactory()
    {
        factories["tea"] = []() {
            auto tea = make_unique<Tea>();
            tea->prepare(200);
            return tea;
        };
        factories["coffee"] = []() {
            auto coffee = make_unique<Coffee>();
            coffee->prepare(50);
            return coffee;
        };
    }

    std::unique_ptr<HotDrink> make_drink(const std::string & name);

private:
    std::map<std::string,
        std::function<std::unique_ptr<HotDrink>()>> factories;
};

inline std::unique_ptr<HotDrink>
    DrinkWithVolumeFactory::make_drink(const std::string & name)
{
    return factories[name]();
}
```

* 사용 예시

```cpp
auto p = PointFactory::NewCartesian(3, 4);
```
