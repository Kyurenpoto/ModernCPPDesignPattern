# 프록시 패턴

## 스마트 포인터

```cpp
struct BankAccount
{
    void deposit(int amount)
    { ... }
};

BankAccount *ba = new BankAccount;
ba->deposit(123);
auto ba2 = std::make_shared<BankAccount>();
ba2->deposit(123);
```

## 속성 프록시

* 구조 예시

```cpp
template<class T>
struct Property
{
    T value;

    Property(const T initial_value)
    {
        *this = initial_value;
    }

    operator T()
    {
        return value;
    }

    T operator =(T new_value)
    {
        return value = new_value;
    }
};
```

* 사용 예시

```cpp
struct Creature
{
    Property<int> strength{10};
    Property<int> agility{5};
};

Creature creature;
creature.agility = 20;
auto x = creature.strength;
```

## 가상 프록시

* 구조 예시

```cpp
struct Image
{
    virtual void draw() = 0;
};

struct Bitmap : Image
{
    Bitmap(const std::string & filename)
    {
        std::cout << "Loading image from " << filename << "\n";
    }

    void draw() override
    {
        std::cout << "Drawing image " << filename << "\n";
    }
};

struct LazyBitmap : Image
{
    LazyBitmap(const std::string & filename) :
        filename{ filename }
    {}

    ~LazyBitmap()
    {
        delete bmp;
    }

    void draw() override
    {
        if (!bmp)   // lazy loading
            bmp = new Bitmap(filename);
        bmp->draw();
    }

private:
    Bitmap *bmp{nullptr};
    std::string filename;
};
```

## 커뮤니케이션 프록시

* 구조 예시

```cpp
struct Pingable
{
    virtual std::wstring ping(const std::wstring & message) = 0;
};

struct Pong : Pingable
{
    std::wstring ping(const wstring & message) override
    {
        return message + L" pong";
    }
};

struct RemotePong : Pingable
{
    std::wstring ping(const wstring & message) override
    {
        std::wstring result;
        http_client client(U("http://localhost:9149/"));
        uri_builder builder(U("/api/pingpong/"));
        builder.append(message);
        pplx::task<std::wstring> task = client.request(
            methods::GET, builder.to_string())
            .then([=](http_response r)
            {
                return r.extract_string();
            });
        task.wait();
        return task.get();
    }
};
```
