# 커맨드 패턴

```cpp
struct BankAccount
{
    int balance = 0;
    int overdraft_limit = -500;

    void deposit(int amount)
    {
        balance += amount;
        std::cout << "deposited: " << amount << ", balance is now " << balance << "\n";
    }

    void withdraw(int amount)
    {
        if (balance - amount >= overdraft_limit)
        {
            balance -= amount;
            std::cout << "withdraw: " << amount << ", balance is now " << balance << "\n";
        }
    }
};
```

## 커맨드 구현

```cpp
struct Command
{
    virtual void call() const = 0;
};

struct BankAccountCommand : Command
{
    BankAccount & account;
    enum Action
    {
        deposit,
        withdraw
    } action;
    int amount;

    BankAccountCommand(BankAccount & account, const Action action, const int amount) :
        account{ account },
        action{ action },
        amount{ amount }
    {}

    void call() const override
    {
        switch (action)
        {
        case deposit:
            account.deposit(amount);
            break;
        case withdraw:
            account.withdraw(amount);
            break;
        }
    }
};
```

## 되돌리기

* 구조 예시

```cpp
struct Command
{
    virtual void call() = 0;
    virtual void undo() = 0;
};

struct BankAccount
{
    ...

    bool withdraw(int amount)
    {
        if (balance - amount >= overdraft_limit)
        {
            balance -= amount;
            std::cout << "withdraw: " << amount << ", balance is now " << balance << "\n";
            return true;
        }
        return false;
    }
};

struct BankAccountCommand : Command
{
    bool withdrawal_succeeded;
    ...

    BankAccountCommand(BankAccount & account, const Action action, const int amount) :
        account{ account },
        action{ action },
        amount{ amount },
        withdrawal_succeeded{ false }
    {}

    void call() const override
    {
        switch (action)
        {
        case deposit:
            account.deposit(amount);
            break;
        case withdraw:
            withdrawal_succeeded = account.withdraw(amount);
            break;
        }
    }

    void undo() override
    {
        switch (action)
        {
        case deposit:
            account.withdraw(amount);
            break;
        case withdraw:
            if (withdrawal_succeeded)
                account.deposit(amount);
            break;
        }
    }
};
```

## 컴포지트 커맨드

* 구조 예시

```cpp
struct CompositeBankAccountCommand :
    std::vector<BankAccountCommand>,
    Command
{
    CompositeBankAccountCommand(const std::initializer_list<std::value_type> & items) :
        std::vector<BankAccountCommand>{ items }
    {}

    void call() override
    {
        for (auto & cmd : *this)
            cmd.call();
    }

    void undo() override
    {
        for (auto it = rbegin(); it != rend(); ++it)
            it->undo();
    }
};
```

## 명령과 조회의 분리

* 구조 예시

```cpp
enum class CreatureAbility
{
    strength,
    agility
};

struct CreatureCommand
{
    enum Action
    {
        set,
        increaseBy,
        decreaseBy
    } action;
    CreatureAbility ability;
    int amount;
};

struct CreatureQuery
{
    CreatureAbility ability;
    int result;
};

struct Creature
{
    Creature(int strength, int agility) :
        strength{ strength },
        agility{ agility }
    {}

    void process_command(const CreatureCommand & c)
    {
        int * ability;
        switch (c.ability)
        {
        case CreatureAbility::strength:
            ability = &strength;
            break;
        case CreatureAbility::agility:
            ability = &agility;
            break;
        }
        switch (c.action)
        {
        case CreatureCommand::set:
            *ability = c.amount;
            break;
        case CreatureCommand::increaseBy:
            *ability += c.amount;
            break;
        case CreatureCommand::decreaseBy:
            *ability -= c.amount;
            break;
        }
    }

    int process_query(const CreatureQuery & q) const
    {
        switch (c.ability)
        {
        case CreatureAbility::strength:
            return strength;
            break;
        case CreatureAbility::agility:
            return agility;
            break;
        }
        return 0;
    }

    void set_strength(int value)
    {
        process_command(CreatureCommand{
            CreatureCommand::set, CreatureCommand::strength, value
        });
    }

    void set_agility(int value)
    {
        process_command(CreatureCommand{
            CreatureCommand::set, CreatureCommand::agility, value
        });
    }

    int get_strength(int value)
    {
        return process_query(CreatureQuery{
            CreatureCommand::strength
        });
    }

    int get_agility(int value)
    {
        return process_query(CreatureQuery{
            CreatureCommand::agility
        });
    }

private:
    int strength, agility;
};
```
