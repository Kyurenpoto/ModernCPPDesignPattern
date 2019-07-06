# 책임 사슬 패턴

```cpp
struct Creature
{
    std::string name;
    int attack, defense;
};
```

## 포인터 사슬

```cpp
struct CreatureModifier
{
    explicit CreatureModifier(Creature & creature) :
        creature{ creature }
    {}

    void add(CreatureModifier * cm)
    {
        if (next)
            next->add(cm);
        else
            next = cm;
    }

    virtual void handle()
    {
        if (next)
            next->handle();
    }

protected:
    Creature & creature;

private:
    CreatureModifier* next{ nullptr };
};
```

## 브로커 사슬

* 구조 예시

```cpp
struct Query
{
    std::string creature_name;
    enum Argument
    {
        attack,
        defense
    } argument;
    int result;
};

struct Game
{
    boost::signal<void(Query &)> queries;
};

struct Creature
{
    std::string name;

    Creature(Game & game, ...) :
        game{ game },
        ...
    {}

    int get_attack() const
    {
        Query q{ name, Query::Argument::attack, attack };
        game.queries(q);
        return q.result;
    }

    int get_defense() const
    {
        Query q{ name, Query::Argument::defense, defense };
        game.queries(q);
        return q.result;
    }

    friend std::ostream & std::ostream::operator << (std::ostream & os, const Creature & c)
    {
        os << c.name << " " << c.attack << " " << c.defense;
        return os;
    }

private:
    Game & game;
    int attack, defense;
};

struct CreatureModifier
{
    CreatureModifier(Game & game, Creature & creature) :
        game{ game },
        creature{ creature }
    {}

private:
    Game & game;
    Creature & creature;
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
