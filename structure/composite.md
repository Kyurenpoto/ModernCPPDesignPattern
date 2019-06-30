# 컴포지트 패턴

## 배열 기반 속성

* 구조 예시

```cpp
struct Creature
{
    int get_strength() const
    {
        return abilities[str];
    }

    void set_strength(int value)
    {
        abilities[str] = value;
    }

    int sum() const
    {
        return std::accumulate(abilities.begin(), abilities.end(), 0);
    }

    double average() const
    {
        return sum() / (double)count;
    }

    int max() const
    {
        return *std::max_element(abilities.begin(), abilities.end());
    }

private:
    enum Abilities
    {
        str,
        agl,
        intl,
        count
    };

    std::array<int, count> abilities;
};
```

## 그래픽 객체의 그루핑

* 구조 예시

```cpp
struct GraphicObject
{
    virtual void draw() = 0;
};

struct Circle : GraphicObject
{
    void draw() override
    {
        std::cout << "Circle\n";
    }
};

struct Group : GraphicObject
{
    std::string name;
    std::vector<GraphicObject*> objects;

    explicit Group(const std::string & name) :
        name{ name }
    {}

    void draw() override
    {
        std::cout << "Group: " << name.c_str() << " contains\n";
        for (auto && o: objects)
            o.draw();
    }
};
```

* 사용 예시

```cpp
Group root("root");
Circle c1, c2;
root.objects.push_back(&c1);

Group subgroup("sub");
subgroup.objects.push_back(&c2);

root.objects.push_back(&subgroup);

root.draw();
```
