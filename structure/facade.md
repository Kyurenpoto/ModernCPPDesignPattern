# 퍼사드 패턴

```cpp
struct TableBuffer : IBuffer
{
    struct TableColumnSpec
    {
        std::string header;
        int width;
        enum class TableColumnAlignment
        {
            Left,
            Center,
            Right
        } alignment;
    };

    TableBuffer(std::vector<TableColumnSpec>& spec, int totalHeight)
    { ... }
};

struct ConsoleCreationParameters
{
    std::optional<Size> client_size;
    int character_width{10};
    int character_height{14};
    int width{20};
    int height{30};
    bool fullscreen{false};
    bool create_default_view_and_buffer{true};
};

struct Console
{
    std::vector<ViewPort*> viewports;
    Size charSize, gridSize;

    Console(bool fullscreen, int char_width, int char_height,
            int width, int height, std::optional<Size> client_size)
    { ... }

    Console(const ConsoleCreationParameters & ccp)
    { ... }

    // ...
};
```
