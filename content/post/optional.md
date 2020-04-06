---
title: "Optional"
date: 2020-04-06T20:11:51+08:00
---

<!--more-->

### std::optional 使用



定义一个简单的结构

```c++
class Entity {
    public:
    explicit Entity(uint64_t id)
        : id_(id) {};
    ~Entity() = default;
    private:
    [[maybe_unused]] uint64_t id_;
};

std::map<uint64_t, Entity> entitys;
```



- find_entity 根据 id 从 entitys 中查找

```c++
Entity find_entity(uint64_t id)
{
    auto it = entitys.find(id);
    if(it == entitys.end())
    {
        return Entity{};  // error
    }
    return it->second;
}
```

`编译错误`

```c++
main.cc:22:16: error: no matching constructor for initialization of 'Entity'
        return Entity{};
               ^     ~~
main.cc:6:14: note: candidate constructor not viable: requires single argument 'id', but no arguments were provided
    explicit Entity(uint64_t id)
             ^
main.cc:4:7: note: candidate constructor (the implicit copy constructor) not viable: requires 1 argument, but 0 were provided
class Entity {
      ^
1 error generated.

```



- 使用 std::optional 作为 find_entity 的返回值

```c++
std::optional<Entity> find_entity(uint64_t id)
{
    auto it = entitys.find(id);
    if (it == entitys.end()) {
        return {};
    }
    return it->second;
}
```



```c++
auto op = find_entity(10);
if (!op) {
    // not found
}
// do something
```


std::optional 需要需要编译器支持c++17，如果编译器不支持，可以使用 boost::optional 或者 absl::optional
••• ~
