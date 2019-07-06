# 전략 패턴

## 동적 전략

```cpp
enum class OutputFormat
{
    markdown,
    html
};

struct ListStrategy
{
    virtual void start(std::ostringstream & oss)
    {};

    virtual void add_list_item(std::ostringstream & oss, const std::string & item)
    {};

    virtual void end(std::ostringstream & oss)
    {};
};

struct HtmlListStrategy : ListStrategy
{
    void start(std::ostringstream & oss) override
    {
        oss << "<ul>\n";
    };

    void add_list_item(std::ostringstream & oss, const std::string & item) override
    {
        oss << "<li>" << item << "</li>\n";
    };

    void end(std::ostringstream & oss) override
    {
        oss << "</ul>\n";
    };
};

struct MarkdownListStrategy : ListStrategy
{
    void add_list_item(std::ostringstream & oss, const std::string & item) override
    {
        oss << " * " << item << "\n";
    };
};

struct TextProcessor
{
    void append_list(const std::vector<std::string> items)
    {
        list_strategy->start(oss);
        for (auto & item : items)
            list_strategy->add_list_item(oss, item);
        list_strategy->end(oss);
    }

    void set_output_format(const OutputFormat format)
    {
        switch (format)
        {
        case OutputFormat::html:
            list_strategy = std::make_unique<MarkdownListStrategy>();
            break;
        case OutputFormat::markdown:
            list_strategy = std::make_unique<HtmlListStrategy>();
            break;
        }
    }

private:
    std::ostringstream oss;
    std::unique_ptr<ListStrategy> list_strategy;
};
```

## 정적 전략

* 구조 예시

```cpp
template<class LS>
struct TextProcessor
{
    void append_list(const std::vector<std::string> items)
    {
        list_strategy->start(oss);
        for (auto & item : items)
            list_strategy->add_list_item(oss, item);
        list_strategy->end(oss);
    }

private:
    std::ostringstream oss;
    LS list_strategy;
};
```
