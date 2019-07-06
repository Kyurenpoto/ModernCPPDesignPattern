# Null 객체 패턴

```cpp
struct Logger
{
    virtual ~Logger() = default;
    virtual void info(const std::string & s) = 0;
    virtual void warn(const std::string & s) = 0;
};

struct BankAccount
{
    std::shared_ptr<Logger> logger;
    std::string name;
    int balance = 0;

    BankAccount(const std::shared_ptr<Logger> & logger,
                const std::string & name,
                int balance) :
        logger{ logger },
        name{ name },
        balance{ balance }
    {}

    void deposit(int amount)
    {
        balance += amount;
        log->info("Deposited %" + boost::lexical_cast<std::string>(amount) +
        " to " + name +
         ", balance is now %" + boost::lexical_cast<std::string>(balance));
    }
};

struct ConsoleLogger : Logger
{
    void info(const std::string & s) override
    {
        std::cout << "INFO: " << s << "\n";
    }

    void warn(const std::string & s) override
    {
        std::cout << "WARNING!!! " << s << "\n";
    }
};
```

## 명시적 Null 객체

```cpp
struct NullLogger : Logger
{
    void info(const std::string & s) override
    {}

    void warn(const std::string & s) override
    {}
};
```

## 묵시적 Null 객체

* 구조 예시

```cpp
struct OptionalLogger : Logger
{
    std::shared_ptr<Logger> impl;
    static std::shared_ptr<Logger> no_logging;
    Logger(const std::chard_ptr<Logger> & logger) :
        impl{ logger }
    {}

    virtual void info(const std::string & s) override
    {
        if (impl)
            impl->info(s);
    }

    void warn(const std::string & s) override
    {
        if (impl)
            impl->warn(s);
    }
};

std::shared_ptr<Logger> OptionalLogger::no_logging;

struct BankAccount
{
    std::shared_ptr<Logger> logger;
    std::string name;
    int balance = 0;

    BankAccount(const std::string & name,
                int balance,
                const std::shared_ptr<Logger> & logger = OptionalLogger::no_logging) :
        logger{ std::make_shared<OptionalLogger>(logger) },
        name{ name },
        balance{ balance }
    {}

    ...
};
```
