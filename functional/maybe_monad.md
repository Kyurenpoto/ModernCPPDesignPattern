# Maybe 모나드 패턴

* 구조 예시

```cpp
template<class T>
struct Maybe
{
    T * context;
    Maybe(T * context) :
        context{ context }
    {}

    template<class Func>
    auto With(Func evaluator)
    {
        return context != nullptr ?
            new Maybe<decltype(evaluator(std::declval<T *>()))>{evaluator(context)} : nullptr;
    }

    template<class Func>
    auto Do(Func action)
    {
        if (context != nullptr)
            action(context);
        return *this;
    }
};
```
