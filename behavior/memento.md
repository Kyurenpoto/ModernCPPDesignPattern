# 메멘토 패턴

## 은행계좌

```cpp
struct Memento
{
        Memento(int balance) :
            balance{ balance }
        {}

        friend class BankAccount;
private:
    int balance;
};

struct BankAccount
{
    explicit BankAccount(const int balance) :
        balance{ balance }
    {}

    Memento deposit(int amount)
    {
        balance += amount;
        return { balance };
    }

    void restore(const Memento & m)
    {
        balance = m.balance;
    }

    friend std::ostream & operator << (std::ostream & os, const BankAccount & ba)
    {
        os << ba.balance;
        return os;
    }

private:
    int balance = 0;
};
```

* 사용 예시

```cpp
void memento()
{
    BankAccount ba{ 100 };
    auto m1 = ba.deposit(50);
    auto m2 = ba.deposit(25);
    std::cout << ba << "\n";

    ba.restore(m1);
    std::cout << ba << "\n";

    ba.restore(m2);
    std::cout << ba << "\n";
}
```

## Undo, Redo

* 구조 예시

```cpp
struct BankAccount
{
    explicit BankAccount(const int balance) :
        balance{ balance }
    {
        changes.emplace_back(std::make_shared<Memento>(balance));
        current = 0;
    }

    std::shared_ptr<Memento> deposit(int amount)
    {
        balance += amount;
        auto m = std::make_shared<Memento>(balance);
        changes.push_back(m);
        ++current;
        return m;
    }

    void restore(const std::shared_ptr<Memento> & m)
    {
        if (m)
        {
            balance = m->balance;
            changes.push_back(m);
            current = changes.size() - 1;
        }
    }

    std::shared_ptr<Memento> undo()
    {
        if (current > 0)
        {
            --current;
            auto m = changes[current];
            balance = m->balance;
            return m;
        }
        return {};
    }

    std::shared_ptr<Memento> redo()
    {
        if (current + 1 < changes.size())
        {
            ++current;
            auto m = changes[current];
            balance = m->balance;
            return m;
        }
        return {};
    }

    friend std::ostream & operator << (std::ostream & os, const BankAccount & ba)
    {
        os << ba.balance;
        return os;
    }

private:
    int balance = 0;
    std::vector<std::shared_ptr<Memento>> changes;
    int current;
};
```

* 사용 예시

```cpp
struct DoubleAttackModifier :
    public CreatureModifier
{
    DoubleAttackModifier(Game & game, Creature & creature) :
        CreatureModifier(game, creature)
    {
        conn = game.queries.connect([&](Query & q)
        {
            if (q.creature_name == creature_name &&
                q.argument == Query::Argument::attack)
                q.result *= 2;
        });
    }

    ~DoubleAttackModifier()
    {
        conn.disconnect();
    }

private:
    boost::connection conn;
};

Game game;
Creature goblin{ game, "Strong Goblin", 2, 2 };
std::cout << goblin << "\n";
{
    DoubleAttackModifier dam{ game, goblin };
    std::cout << goblin << "\n";
}
std::cout << goblin << "\n";
```
