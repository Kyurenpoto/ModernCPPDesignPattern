# 프로토타입 패턴

* 문제 상황

```cpp
Contract john{ "John Doe", Address{ "123 East Dr", "London", 10 } };
Contract jane{ "Jane Doe", Address{ "123 East Dr", "London", 11 } };
```

## 평범한 중복 처리

* 구조 예시

```cpp
struct Address
{
    std::string street, city;
    int suite;
};

struct Contract
{
    string name;
    Address addresss;
};

Contract worker{ "", Address{ "123 East Dr", "London", 0 } };
```

* 사용 예시

```cpp
Contract john = worker;
john.name = "John Doe";
john.address.suite = 10;

Contract jane = worker;
jane.name = "Jane Doe";
jane.address.suite = 11;
```

## 복제 생성자를 통한 중복 처리

* 구조 예시

```cpp
struct Address
{
    std::string street, city;
    int suite;

    Address(const std::string & street,
            const std::string & city,
            const int suite) :
        street{ street },
        city{ city },
        suite{ suite }
    {}
};

template<class T>
struct Cloneable
{
    virtual T clone() const = 0;
};

struct Contract : Cloneable<Contract>
{
    string name;
    Address * addresss;

    Contract clone() const override
    {
        return Contract{ name, new Address{ *other.address } };
    }
};

Contract worker{ "", new Address{ "123 East Dr", "London", 0 } };
```

* 사용 예시

```cpp
Contract john = worker.clone();
john.name = "John Doe";
john.address->suite = 10;

Contract jane = worker.clone();;
jane.name = "Jane Doe";
jane.address->suite = 11;
```

## 직렬화

* 구조 예시

```cpp
struct Address
{
    std::string street, city;
    int suite;

private:
    friend class boost::serialization::access;
    template<class Ar> void serialize(Ar & ar, const unsigned int version)
    {
        ar & street;
        ar & city;
        ar & suite;
    }
};

struct Contract
{
    string name;
    Address * addresss = nullptr;

private:
    friend class boost::serialization::access;
    template<class Ar> void serialize(Ar & ar, const unsigned int version)
    {
        ar & name;
        ar & address;
    }
};

Contract clone(const Contract & c)
{
    std::ostringstream oss;
    boost::archive::text_oarchive oa{ oss };
    oa << c;
    string s = oss.str();

    istringstream iss{ oss.str() };
    boost::archive::text_iarchive ia{ iss };
    Contract result;
    ia >> result;
    return result;
}
```

* 사용 예시

```cpp
Contract john{ "John Doe", Address{ "123 East Dr", "London", 10 } };
Contract jane = clone(john);
jane.name = "Jane Doe";
jane.address->suite = 11;
```

## 프로토타입 팩토리

* 구조 예시

```cpp
struct EmployeeFactory
{
    static Contract main;
    static Contract aux;

    static std::unique_ptr<Contract>
        NewMainOfficeEmployee(std::string name, int suite)
    {
        return NewEmployee(name, suite, main);
    }

    static std::unique_ptr<Contract>
        NewAuxOfficeEmployee(std::string name, int suite)
    {
        return NewEmployee(name, suite, aux);
    }

private:
    static std::unique_ptr<Contract>
        NewEmployee(std::string name, int suite, Contract & proto)
    {
        auto result = make_unique<Contract>(proto);
        result->name = name;
        result->address->suite = suite;
        return result;
    }
};
```

* 사용 예시

```cpp
auto john = EmployeeFactory::NewAuxOfficeEmployee("John Doe", 123);
auto jane = EmployeeFactory::NewMainOfficeEmployee("Jane Doe", 125);
```
