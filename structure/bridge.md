# 브릿지 패턴

## Pimpl

* 구조 예시

```cpp
struct Person
{
    std::string name;
    void greet();

    Person();
    ~Person();

    struct PersonImpl;

    PersonImpl * impl;
};

struct Person::PersonImpl
{
    void greet(Person * p);
};

Person::Person() :
    impl{ new PersonImpl }
{}

Person::~Person()
{
    delete impl;
}

void Person::greet()
{
    impl->greet(this);
}

void Person::PersonImpl::greet(Person * p)
{
    printf("hello %s", p->name.c_str());
}
```

* 사용 예시

```cpp
std::vector<std::shared_ptr<VectorObject>> vectorObjects{
    std::make_shared<VectorRectangle>(10, 10, 100, 100),
    std::make_shared<VectorRectangle>(30, 30, 60, 60)
};

for (auto & obj : vectorObjects)
{
    for (auto & line : *obj)
    {
        LineToPointAdapter lpo{ line };
        DrawPoints(dc, lpo.begin(), lpo.end());
    }
}
```

## 브릿지

* 구조 예시

```cpp
struct Renderer
{
    virtual void render_circle(float x, float y, float radius) = 0;
};

struct VectorRenderer : Renderer
{
    void render_circle(float x, float y, float radius) override
    {
        cout << "Drawing a vector circle of radius " << radius << "\n";
    }
};

struct RasterRenderer : Renderer
{
    void render_circle(float x, float y, float radius) override
    {
        cout << "Rasterizing circle of radius " << radius << "\n";
    }
};

struct Shape
{
    virtual void draw() = 0;
    virtual void resize(float factor) = 0;

protected:
    Renderer & renderer;
    Shape(Renderer & renderer) :
        renderer{ renderer }
    {}
};

struct Circle : Shaoe
{
    float x, y, radius;

    void draw() override
    {
        renderer.render_circle(x, y, radius);
    }

    void resize(float factor) override
    {
        radius *= factor;
    }

    Circle(Renderer & renderer, float x, float y, float radius) :
        Shape{ renderer }, x{ x }, y{ y }, radius{ radius }
    {}
};
```

* 사용 예시

```cpp
RasterRenderer rr;
Circle raster_circle{ rr, 5, 5, 5 };
raster_circle.draw();
raster_circle.resize(2);
raster_circle.draw();
```
