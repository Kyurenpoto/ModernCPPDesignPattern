# 관찰자 패턴

## Observer`<T>`

```cpp
template<class T>
struct Observer
{
    virtual void field_changed(T & source, const std::string & field_name) = 0;
};
```

## Observable`<T>`

* 구조 예시

```cpp
template<class T>
struct Obervable
{
    void notify(T & source, const std::string & name)
    {
        for (auto obs : observers)
            obs->field_changed(source, name);
    }

    void subscribe(Observer<T> * f)
    {
        observers.push_back(f);
    }

    void unsubscribe(Observer<T> * f);

private:
    std::vector<Observer<T> *> observers;

    // T의 setter에서 notify()가 호출되어야 함
};
```

## 스레드 안정성

* 구조 예시

```cpp
template<class T>
struct Obervable
{
    void notify(T & source, const std::string & name)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        for (auto obs : observers)
            obs->field_changed(source, name);
    }

    void subscribe(Observer<T> * f)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        observers.push_back(f);
    }

    void unsubscribe(Observer<T> * f)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

private:
    std::vector<Observer<T> *> observers;
    std::mutex mtx;
};
```

## 재진입성

* 구조 예시

```cpp
template<class T>
struct Obervable
{
    void notify(T & source, const std::string & name)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        for (auto obs : observers)
            obs->field_changed(source, name);
    }

    void subscribe(Observer<T> * f)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        observers.push_back(f);
    }

    void unsubscribe(Observer<T> * f)
    {
        std::scoped_lock<std::mutex> lock{ mtx };

        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

private:
    std::vector<Observer<T> *> observers;
    std::recursive_mutex mtx;
};
```

## Boost.Signals2

* 구조 예시

```cpp
template<class T>
struct Obervable
{
    boost::signal<void(T &, const std::string &)> property_changed;
};
```
