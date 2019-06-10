# 어댑터 패턴

* 문제 상황

```cpp
struct Point
{
    int x, y;
};

struct Line
{
    Point start, end;
};

struct VectorObject
{
    virtual std::vector<Line>::iterator begin() = 0;
    virtual std::vector<Line>::iterator end() = 0;
};

struct VectorRectangle : VectorObject
{
    VectorRectangle(int x, int y, int width, int height)
    {
        lines.emplace_back(Line({ Point{x, y},
                                  Point{x + width, y} }));
        lines.emplace_back(Line({ Point{x + width, y},
                                  Point{x + width, y + height} }));
        lines.emplace_back(Line({ Point{x + width, y + height},
                                  Point{x, y + height} }));
        lines.emplace_back(Line({ Point{x, y + height},
                                  Point{x, y} }));
    }

    std::vector<Line>::iterator begin() override
    {
        return lines.begin();
    }

    std::vector<Line>::iterator end() override
    {
        return lines.end();
    }

private:
    std::vector<Line> lines;
};

void DrawPoints(CPaintDC & dc, std::vector<Point>::iterator start,
                               std::vector<Point>::iterator end)
{
    for (auto i = start; i != end; ++i)
        dc.SetPixel(i->x, i->y, 0);
}
```

## 어댑터

* 구조 예시

```cpp
struct LineToPointAdapter
{
    using Points = std::vector<Point>;

    LineToPointAdapter(Line & line)
    {
        int left = min(line.start.x, ilne.end.x);
        int right = max(line.start.x, ilne.end.x);
        int top = min(line.start.y, ilne.end.y);
        int bottom = max(line.start.y, ilne.end.y);
        int dx = right - left;
        int dy = bottom - top;

        if (dx == 0)
        {
            for (int y = top; y <= bottom; ++y)
            {
                points.emplace_back(Point{ left, y });
            }
        }
        else if(dy == 0)
        {
            for (int x = left; x <= right; ++x)
            {
                points.emplace_back(Point{ x, top });
            }
        }
    }

    virtual Points::iterator begin()
    {
        return points.begin();
    }

    virtual Points::iterator end()
    {
        return points.end();
    }

private:
    Points points;
};
```

* 사용 예시

```cpp
std::vector<std::shared_ptr<VectorObject>> vectorObjects{
    std::make_shared<VectorRectangle>(10, 10, 100, 100),
    std::make_shared<VectorRectangle>(30, 30, 60, 60)
};

for (auto & obj : vectorObjects)
{
    for (auto & line : *obj)
    {
        LineToPointAdapter lpo{ line };
        DrawPoints(dc, lpo.begin(), lpo.end());
    }
}
```

## 일시적 어댑터

* 구조 예시

```cpp
struct Point
{
    int x, y;

    friend std::size_t hash_value(const Point & obj)
    {
        std::size_t seed = 0x725C686F;
        boost::hash_combine(seed, obj.x);
        boost::hash_combine(seed, obj.y);
        return seed;
    }
};

struct Line
{
    Point start, end;

    friend std::size_t hash_value(const Line & obj)
    {
        std::size_t seed = 0x719C6B16;
        boost::hash_combine(seed, obj.start);
        boost::hash_combine(seed, obj.end);
        return seed;
    }
};

struct LineToPointCachingAdapter
{
    using Points = std::vector<Point>;

    LineToPointCachingAdapter(Line & line)
    {
        static boost::hash<Line> hash;
        line_hash = hash(line);
        if (cache.find(line_hash) != cache.end())
            return;

        Points points;

        int left = min(line.start.x, ilne.end.x);
        int right = max(line.start.x, ilne.end.x);
        int top = min(line.start.y, ilne.end.y);
        int bottom = max(line.start.y, ilne.end.y);
        int dx = right - left;
        int dy = bottom - top;

        if (dx == 0)
        {
            for (int y = top; y <= bottom; ++y)
            {
                points.emplace_back(Point{ left, y });
            }
        }
        else if(dy == 0)
        {
            for (int x = left; x <= right; ++x)
            {
                points.emplace_back(Point{ x, top });
            }
        }

        cache[line_hash] = points;
    }

    virtual Points::iterator begin()
    {
        return cache[line_hash].begin();
    }

    virtual Points::iterator end()
    {
        return cache[line_hash].end();
    }

private:
    std::size_t line_hash;
    static map<size_t, Points> cache;
};
```

* 사용 예시

```cpp
std::vector<std::shared_ptr<VectorObject>> vectorObjects{
    std::make_shared<VectorRectangle>(10, 10, 100, 100),
    std::make_shared<VectorRectangle>(30, 30, 60, 60)
};

std::vector<Point> points;

for (auto & obj : vectorObjects)
{
    for (auto & line : *obj)
    {
        LineToPointCachingAdapter lpo{ line };
        for (auto & p : lpo)
            points.push_back(p);
    }
}

DrawPoints(dc, points.begin(), points.end());
```
