## Elixir Basic

### Style Guide

[The Elixir Style Guide](https://github.com/niftyn8/elixir_style_guide)

### 正则：

```elixir
Regex.run ~r{[aeiou]}, ​"caterpillar"​
Regex.scan ~r{[aeiou]}, ​"caterpillar"​
Regex.split ~r{[aeiou]}, ​"caterpillar"​
Regex.replace ~r{[aeiou]}, ​"caterpillar"​, ​"*"​
```

### 集合类型

#### Tuple

`{1, 2}`

#### List

```elixir
[1, 2, 3] ++ [4, 5, 6] # [1,2,3,4,5,6]
1 in [1,2,3,4] # true
```

#### Keyword List

```elixir
[name: ​"Dave"​, city: ​"Dallas"​, likes: ​"Programming"​]
```

其实是 tuple 的 list

```elixir
[{:name, ​"Dave"​}, {:city, ​"Dallas"​}, {:likes, ​"Programming"​}]
```

#### Map

```elixir
states = %{"AL" => "Alabama", "WI" => "Wisconsin"}
states["AL"]
```

如果 key 是 atom 的话，可以直接使用 dot notation：

```elixir
colors = %{red: 0xff0000, green: 0x00ff00, blue: 0x0000ff}
colors.red # 16711680
```

#### & notation

使用 & notation 创建匿名函数：

```elixir
add_one = &(&1 + 1) ​# same as add_one = fn (n) -> n + 1 end​
sum = &(&1 + &2)
```