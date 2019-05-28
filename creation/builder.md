# 빌더 패턴

* 생성이 까다로운 객체에 적용
* 예시

```cpp
struct HtmlElement
{
    std::string name;
    std::string text;
    std::vector<HtmlElement> elements;

    HtmlElement() = default;
    HtmlElement(const std::string & name, const std::string & text) :
        name{ name },
        text{ text }
    {}
    
    std::string str(int indent = 0) const
    {
        // 컨텐츠를 양식에 맞추어 출력
    }
};
```

## 단순 빌더

* 구조 예시

```cpp
struct HtmlBuilder
{
    HtmlElement root;

    HtmlBuilder(const std::string root_name)
    {
        root.name = root_name;
    }

    void add_child(const std::string child_name, const std::string child_text)
    {
        root.elements.emplace_back(HtmlElement{ child_name, child_text });
    }

    std::string str()
    {
        return root.str();
    }
};
```

* 사용 예시

```cpp
HtmlBuilder builder{ "ui" };
builder.add_child("li", "hello");
builder.add_child("li", "world");
cout << builder.str() << endl;
```

## 흐름식 빌더

* 구조 예시

```cpp
struct HtmlBuilder
{
    // 생략

    HtmlBuilder & add_child(const std::string child_name, const std::string child_text)
    {
        root.elements.emplace_back(HtmlElement{ child_name, child_text });
        return *this;
    }

    // 생략
};
```

* 사용 예시

```cpp
HtmlBuilder builder{ "ui" };
builder.add_child("li", "hello").add_child("li", "world");
cout << builder.str() << endl;
```

### 의도 알려주기

* 빌더 사용을 강제하기

```cpp
struct HtmlBuilder;

struct HtmlElement
{
    // 생략

    const std::size_t indent_size = 2;

    static unique_ptr<HtmlBuilder> build(const std::string & root_name)
    {
        return make_unique<HtmlBuilder>(root_name);
    }
    
    std::string str(int indent = indent_size) const
    {
        // 컨텐츠를 양식에 맞추어 출력
    }

protected:
    HtmlElement() = default;
    HtmlElement(const std::string & name, const std::string & text) :
        name{ name },
        text{ text }
    {}
};

struct HtmlBuilder
{
    // 생략

    operator HtmlElement() const
    {
        return root;
    }

    HtmlElement build() const
    {
        return root;
    }
};
```

* 사용 예시

```cpp
HtmlElement e = HtmlElement::build("ui")
    .add_child("li", "hello")
    .add_child("li", "world");
cout << e.str() << endl;
```

## 그루비 스타일 빌더

* DSL 구현 예시

```cpp
struct Tag
{
    std::string name;
    std::string text;
    std::vector<Tag> children;
    std::vector<std::pair<std::string, std::string>> attributes;

    friend std::ostream & operator << (std::ostream & os, const Tag & tag)
    {
        // 적절하게 구현
    }

protected:
    Tag(const std::string & name, const std::string & text) :
        name{ name },
        text{ text }
    {}
    Tag(const std::string & name, const std::vector<Tag> & children) :
        name{ name },
        children{ children }
    {}
};

struct P : Tag
{
    explicit P(const std::string  & text) :
        Tag{ "p", text }
    {}
    P(std::initializer_list<Tag> children) :
        Tag{ "p", children }
    {}
};

struct IMG : Tag
{
    explicit IMG(const std::string & url) :
        Tag{ "img", "" }
    {
        attributes.emplace_back({ "src", url });
    }
};
```

* 사용 예시

```cpp
std::cout <<
    P {
        IMG { "http://pokemon.com/pikachu.png" }
    }
    << std::endl;
```

## 컴포지트 빌더

* 객체 하나를 생성하는데 복수의 빌더가 사용되는 경우

```cpp
struct PersonBuilder;

class Person
{
    std::string street_address, post_code, city;

    std::string company_name, position;
    int annual_income = 0;

    Person() = default;

public:
    static PersonBuilder create()
    {
        return PersonBuilder{};
    }
};

struct PersonAddressBuilder;
struct PersonJobBuilder;

struct PersonBuilderBase
{
    operator Person()
    {
        return std::move(person);
    }

    PersonAddressBuilder lives() const;
    PersonJobBuilder works() const;

protected:
    Person & person;

    explicit PersonBuilderBase(Person & person) :
        person{ person }
    {}
};

struct PersonBuilder : public PersonBuilderBase
{
    PersonBuilder() :
        PersonBuilderBase{ p }
    {}

private:
    Person p;
};

class PersonAddressBuilder : public PersonBuilderBase
{
    using self = PersonAddressBuilder;

public:
    explicit PersonAddressBuilder(Person & person) :
        PersonBuilderBase{ person }
    {}

    self & at(std::string street_address)
    {
        person.street_address = street_address;
        return *this;
    }

    self & with_postcode(std::string post_code)
    {
        person.post_code = post_code;
        return *this;
    }

    self & in(std::string city)
    {
        person.city = city;
        return *this;
    }
};

class PersonJobBuilder : public PersonBuilderBase
{
    using self = PersonJobBuilder;

public:
    explicit PersonJobBuilder(Person & person) :
        PersonBuilderBase{ person }
    {}

    self & at(std::string company_name)
    {
        person.company_name = company_name;
        return *this;
    }

    self & as_a(std::string position)
    {
        person.position = position;
        return *this;
    }

    self & earning(int annual_income)
    {
        person.annual_income = annual_income;
        return *this;
    }
};

```

* 사용 예시

```cpp
Person p = Person::create()
    .lives().at("123 London Load")
            .with_postcode("SW1 1GB")
            .in("London")
    .works().at("PragmaSoft")
            .as_a("Consultant")
            .earning(10e6);
```
