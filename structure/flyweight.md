# 플라이웨이트 패턴

## 수작업 플라이웨이트

```cpp
using key = uint32_t;

struct User
{
    User(const std::string & first_name,
         const std::string & last_name) :
        first_name{ add(first_name) },
        last_name{ add(last_name) }
    {}

    const std::string & get_first_name() const
    {
        return names.left.find(first_name)->second;
    }

    const std::string & get_last_name() const
    {
        return names.left.find(last_name)->second;
    }

    friend std::ostream & operator<< (std::ostream & os, const User & obj)
    {
        return os
            << "first_name: " << obj.get_first_name()
            << "last_name: " << obj.get_last_name();
    }

protected:
    key first_name, last_name;
    static boost::bimap<key, std::string> names;
    static key seed;
    static key add(const std::string & s)
    {
        auto it = names.right.find(s);
        if (it == names.right.end())
        {
            names.insert({++seed, s});
            return seed;
        }
        return it->second;
    }
};
```

## Boost.Flyweight

* 구조 예시

```cpp
struct User
{
    boost::flyweight<std::string> first_name, last_name;

    User(const std::string & first_name, const std::string & last_name) :
        first_name{ first_name },
        last_name{ last_name }
    {}
};
```

* 사용 예시

```cpp
User john_doe{"John", "Doe"};
User jane_doe{"Jane", "Doe"};
std::cout << (&jane_doe.last_name.get() == &john_doe.last_name.get());
```

## 플라이웨이트 구현

* 구조 예시

```cpp
struct BetterFormattedText
{
    struct TextRange
    {
        int start, end;
        bool capitalize;

        bool covers(int position) const
        {
            return postion >= start && position <= end;
        }
    };

    TextRange & get_range(int start, int end)
    {
        formatting.emplace_back(TextRange{start, end});
        return *formatting.rbegin();
    }

    fritend std::ostream & operator<< (std::ostream & os, const BetterFormattedText & obj)
    {
        std::string s;
        for (size_t i = 0; i < obj.plain_text.length(); ++i)
        {
            auto c = obj.plain_text[i];
            for (const auto & rng : obj.formatting)
            {
                if (rng.covers(i) && rng.capitalize)
                    c = toupper(c);
                s += c;
            }
        }
        return os << s;
    }

private:
    std::string plain_text;
    std::vector<TextRange> formatting;
};
```

* 사용 예시

```cpp
BetterFormattedText bft{"This is a brave new world"};
bft.get_range(10, 15).capitalize = true;
std::cout << bft << "\n";
```
