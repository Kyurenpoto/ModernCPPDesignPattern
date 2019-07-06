# 매개자 패턴

## 채팅룸

```cpp
struct ChatRoom;

struct Person
{
    std::string name;
    ChatRoom * room = nullptr;
    std::vector<std::string> chat_log;

    Person(const std::string & name) :
        name{ name }
    {}

    void receive(const std::string & origin, const std::string & message)
    {
        std::string s{ origin + ": \"" + message + "\"" };
        std::cout << "[" << name << "'s chat session] " << s << "\n";
        chat_log.emplace_back(s);
    }

    void say(const std::string & message) const
    {
        room->broadcast(name, message);
    }

    void pm(const std::string & who, const std::string & message) const
    {
        room->message(name, who, message);
    }
};

struct ChatRoom
{
    std::vector<Person *> people;

    void join(Person * p)
    {
        std::string join_msg = p->name + " joins the chat";
        broadcast("room", join_msg);
        p->room = this;
        people.push_back(p);
    }

    void broadcast(const std::string & origin, const std::string & message)
    {
        for (auto p : people)
            if (p->name != origin)
                p->receive(origin, message);
    }

    void message(const std::string & origin, const std::string & who, const std::string & message)
    {
        auto target = find_if(people.begin(), people.end(),
            [&](const Person * p)
            {
                return p->name == who;
            });
        if (target != people.end())
            (*target)->receive(origin, message);
    }
};
```

## 이벤트

* 구조 예시

```cpp
struct EventData
{
    virtual ~EventData() = default;
    virtual void print() const = 0;
};

struct PlayerScoredData : EventData
{
    std::string player_name;
    int goals_scored_so_Far;

    PlayerScoredData(const std::string & player_name, const int goals_scored_so_far) :
        player_name{ player_name },
        goals_scored_so_far{ goals_scored_so_far }
    {}

    void print() const override
    {
        std::cout << player_name << " has scored! (their " << goals_scored_so_far << " goal)\n";
    }
};

struct Game
{
    boost::signal<void(EventData *)> events;
};

struct Player
{
    std::string name;
    int goals_scored = 0;
    Game & game;

    Player(const std::string & name, Game & game) :
        name{ name },
        game[ game ]
    {}

    void score()
    {
        ++golas_scored;
        PlayerScoredData ps{name, goals_scored};
        game.events(&ps);
    }
};

struct Coach
{
    Game & game;

    explicit Coach(Game & game) :
        game{ game }
    {
        game.events.connect([](EventData * e)
        {
            PlayerScoredData * ps = dynamic_cast<PlayerScoredData *>(e);
            if (ps && ps->goals_scored_so_far < 3)
                std::cout << "coach says: well done, " << ps->player_name << "\n";
        });
    }
};
```
