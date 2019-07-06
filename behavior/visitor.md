# 방문자 패턴

```cpp
struct Expression
{};

struct DoubleExpression : Expression
{
    double value;
    explicit DoubleExpression(const double value) :
        value{ value }
    {}
};

struct AdditionExpression : Expression
{
    Expression *left, *right;

    AdditionExpression(Expression * const left, Expression * const right) :
        left{ left },
        right{ right }
    {}

    ~AdditionExpression()
    {
        delete left;
        delete right;
    }
};
```

## 침습적 방문자

```cpp
struct Expression
{
    virtual void print(std::ostringstream & oss) = 0;
};

struct AdditionExpression : Expression
{
    ...

    void print(std::ostringstream & oss) override
    {
        oss << "(";
        left->print(oss);
        oss << "+";
        right->print(oss);
        oss << ")";
    }
};
```

## 반응형 출력

* 구조 예시

```cpp
struct Expression
{
    virtual ~Expression() = default;
};

struct ExpressionPrinter
{
    void print(Expression * e) const
    {
        if (auto de = dynamic_cast<DoubleExpression *>(e))
        {
            oss << de->value;
        }
        else if (auto ae = dynamic_cast<AdditionExpression *>(e))
        {
            oss << "(";
            print(ae->left, oss);
            oss << "+";
            print(ae->right, oss);
            oss << ")";
        }
    }

    std::string str() const
    {
        return oss.str();
    }

private:
    std::ostringstream oss;
};
```

## 전통적인 방문자(이중 디스패치)

* 구조 예시

```cpp
struct ExpressionVisitor;

struct Expression
{
    virtual void accept(ExpressionVisitor * visitor) = 0;
};

struct DoubleExpression : Expression
{
    ...

    void accept(ExpressionVisitor * visitor) override
    {
        visitor->visit(this);
    }
};

struct AdditionExpression : Expression
{
    ...

    void accept(ExpressionVisitor * visitor) override
    {
        visitor->visit(this);
    }
};

struct ExpressionVisitor
{
    virtual void visit(DoubleExpression * de) = 0;
    virtual void visit(AdditionExpression * ae) = 0;
};

struct ExpressionPrinter : ExpressionVisitor
{
    std::ostringstream oss;

    std::string str() const
    {
        return oss.str();
    }

    void visit(DoubleExpression * de) override
    {
        oss << de->value;
    }

    void visit(AdditionExpression * ae) override
    {
        oss << "(";
        print(ae->left, oss);
        oss << "+";
        print(ae->right, oss);
        oss << ")";
    }
};
```

## 비순환 방문자

* 구조 예시

```cpp
template<class Visitable>
struct Visitor
{
    virtual void visit(Visitable & obj) = 0;
};

struct VisitorBase
{
    virtual ~VisitorBase() = default;
};

struct Expression
{
    virtual ~Expression() = default;

    virtual void accept(VisitorBase & obj)
    {
        if (auto ev = dynamic_cast<Visitor<Expression> *>(&obj))
            ev->visit(*this);
    }
};

struct ExpressionPrinter :
    VisitorBase,
    Visitor<DoubleExpression>,
    Visitor<AdditionExpression>
{
    void visit(DoubleExpression & obj) override;
    void visit(AdditionExpression & obj) override;

    std::string str() const
    {
        return oss.str();
    }

private:
    std::ostringstream oss;
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
