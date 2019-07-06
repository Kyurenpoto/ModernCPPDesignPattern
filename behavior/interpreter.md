# 인터프리터 패턴

## Boost.Spirit

### 추상 구문 트리

```cpp
struct ast_elemnet
{
    virtual ~ast_element() = default;
    virtual void accept(ast_element_visitor & visitor) = 0;
};

struct property : ast_element
{
    std::vector<wstring> names;
    type_specification type;
    bool is_constant{ false };
    std::wstring default_value;

    void accept(ast_element_visitor & visitor) override
    {
        visitor.visit(*this);
    }
};

BOOST_FUSION_ADAPT_STRUCT(
    tl n::property,
    (std::vector<std::wstring>, names),
    (tl n::type_specification, type),
    (bool, is_constant),
    (std::wstring, default_value),
);

using class_member = std::varient<function_body, property, function_signature>;
```

### 파서

```cpp
class_declaration_rule %=
    lit(L"class ") >> +(alnum) % '.'
    >> -(lit(L"(") >> -parameter_declaration_rule % ',') >> lit(L")"))
    >> "{"
    >> *(function_body_rule | property_rule | function_signature_rule)
    >> "}";

template<class TLanguagePrinter, class Iterator>
std::wstring parse(Iterator first, Iterator last)
{
    using boost::spirit::qi::phrase_parse;

    file f;
    file_parse<std::wstring::const_iterator> fp{};
    auto b = phrase_parse(first, last, fp, spase, f);
    if (b)
        return TLanguagePrinter{}.prettyprint(f);
    return std::wstring(L"FAIL");
}
```

### 프린터

```cpp
struct default_value_visitor : static_visitor<>
{
    cpp_printer & printer;

    explicit default_value_visitor(cpp_printer & printer) :
        printer{ printer }
    {}

    void operator() (const basic_type & bt) const
    {
        printer.buffer << printer.default_value_for(bt.name);
    }

    void operator() (const tuple_signature & ts) const
    {
        for (auto & e : ts.elements)
        {
            (*this)(e.type);
            printer.buffer << ", ";
        }
        printer.backtrack(2);
    }
};
```
