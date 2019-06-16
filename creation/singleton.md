# 싱글턴 패턴

## 전통적인 구현

* 구조 예시

```cpp
struct Database
{
    static Database & get()
    {
        // C++11 이상에서는 Thread-Safe
        static Database * database = new Database;
        return *database;
    }
    Database(const Database &) = delete;
    Database(Database &&) = delete;
    Database & operator = (const Database &) = delete;
    Database & operator = (Database &&) = delete;

protected:
    Database() { /*...*/ }
};
```

## 싱글턴의 의존성 문제 해결

* 구조 예시

```cpp
struct Database
{
    virtual int get_population(const std::string & name) = 0;
};

struct SingletonDatabase : Database
{
    SingletonDatabase(const SingletonDatabase &) = delete;
    SingletonDatabase & operator = (const SingletonDatabase &) = delete;
    
    static SingletonDatabase & get()
    {
        static SingletonDatabase db;
        return db;
    }

    int get_population(const std::string & name) override
    {
        return capitals[name];
    }

private:
    SingletonDatabase() {}

    std::map<std::string, int> capitals;
};

struct ConfigurableRecordFinder
{
    explicit ConfigurableRecordFinder(Database & db) :
        db{ db }
    {}

    int total_population(std::vector<std::string> names)
    {
        int result = 0;
        for (auto & name : names)
            result += db.get().get_population(name);
        return result;
    }

    Database & db;
};
```

* 사용 예시

```cpp
struct DummyDatabase : Database
{
    DummyDatabase()
    {
        capitals["alpha"] = 1;
        capitals["beta"] = 2;
        capitals["gamma"] = 3;
    }

    int get_population(const std::string & name) override
    {
        return capitals[name];
    }

private:
    std::map<std::string, int> capitals;
};

TEST(RecordFinderTests, DummyTotalPopulationTest)
{
    DummyDatabase db;
    ConfigurableRecordFinder rf{ db };
    EXPECT_EQ(4, rf.total_population(std::vector<std::string>{"alpha", "gamma"}));
}
```

## 제어 역전

* 사용 예시

```cpp
auto injector = di::make_injector(di::bind<IFoo>.to<Foo>.in(di::singleton), ...);
```

## 모노스테이트

* 구조 예시

```cpp
struct Printer
{
    int get_id() const
    {
        return id;
    }

    void set_id(int value)
    {
        id = value;
    }

private:
    static int id;
};
```
