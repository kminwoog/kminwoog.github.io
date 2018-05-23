---
title:  "What's struct in elixir"
categories: 
  - elixir
last_modified_at: 2018-05-09T11:54:59-09:00
tags: elixir elixirlang tutorial
comments: true
---

엘릭서에서 `struct`는 기본적으로 `map`과 유사하다.

맵과 동일하게 `key`와 `value`를 가지고 기본값(default)을 가질 수 있으며, [Strict programming language](https://en.wikipedia.org/wiki/Strict_programming_language)를 경험하던 사람은 때론 느슨하게 때론 불안하게 느낄 수 있는 부족함을 메꾸어 주는 compile type-checking 기능을 제공한다.

elixir 를 처음 사용하는 사람들은 아마 `ecto`를 사용하다 `struct`를 접하지 않을까? 아직 숙련되지 않은 상황에서는 `map`의 기능을 숙지한 후 차이를 알아가는 것이 좀 더 이해하기 쉬울지도 모르겠다.

## struct의 활용
정의에서 알 수 있듯이 말 그대로 기본값을 가진 struct를 생성하기 쉽다. struct의 맴버들이 많을 때 추가된 맴버를 일일이 초기화하려면 번거롭고 초기값을 미리 지정해두면 실수도 줄어들지 않을까. map에 key를 가졌는지 `Map.has_key?/2`를 호출하거나 `map[:no_exist_field] == nil` 와 같은 지저분한 코드를 줄일 수 있다.

```elixir
defmodule Player do
  defstruct name: "unassigned",
            max_hp: 0

  # 기본값 정의
  def new(char \\ %Player{})

  # 1. 인자가 Player struct인 pattern mactching
  def new(char = %Player{}) do
    char
  end

  # 2. 인자가 map인 pattern matching
  def new(init_char) when is_map(init_char) do
    struct(Player, init_char)
  end
end
```

1. new()함수에 인자를 넘기지 않으면 기본적으로 `기본값을 가진 struct`를 생성하여 그대로 반환한다.
2. 인자로 넘어온 값이 map이면 `struct/2`를 통해 `Map.merge(%Player{}, init_char)` 형태로 수행된다.
`Map.merge/2`는 왼쪽 맵이 struct이든 아니든 오른쪽 맵의 값이 merge 된다. 왼쪽 struct의 key가 `존재하는 것만` merge하기 위해선 위의 코드와 같이 `struct/2`를 사용해야 한다

```sh
iex> Player.new()
%Player{max_hp: 0, name: "unassigned"}
iex> pc1 = Player.new(%{max_hp: 1000, name: "kminwoog"})
%Player{max_hp: 1000, name: "kminwoog"}
iex> pc1 == Player.new(%Player{max_hp: 1000, name: "kminwoog"})
true
iex> Player.new(%{unknown: 4444})
%Player{max_hp: 0, name: "unassigned"}
iex> Player.new(%Player{unknown: 4444})
** (KeyError) key :unknown not found in: %Player{max_hp: 0, name: "unassigned"}
...
```

오랫동안 변수에 타입을 정의하는 언어를 사용해 왔기 때문에 
> 가능한한 많은 곳에 struct를 사용하는 것이 좋지 않을까?

언제나 그랬던 것 처럼 코드 상황에 따라 변수가 있겠지만 `struct의 불 필요한 맴버(필드)의 제어가 가능`하다면 `맴버가 일정 개수(20?) 이하`이면 struct로 정의하는 것이 좋을 것 같다.


## Merge
`map`을 머지할 때 `Map.merge/2` 보다 `%{map | name: "new_name"}` 구문을 사용하면 좀더 직관적으로 코드가 보인다.
`struct`도 동일하게 `%Player{pc1 | name: "nimo"}` 형태로 사용 가능하다.  
단, `pc1`은 struct여야 한다.

## 내부 들여다 보기

`Map.from_struct/1`를 보면 `Map.delete(struct, :__struct__)`를 수행한다.
struct는 `__struct__`라는 필드에 `defstruct가 정의된 모듈`을 가지는 특수한 형태의 map이라고 정의할 수 있다.

```sh
iex> Map.from_struct(pc1)
%{max_hp: 1000, name: "kminwoog"}
iex> Map.delete(pc1, :__struct__)
%{max_hp: 1000, name: "kminwoog"}
iex> Map.keys(pc1)
[:__struct__, :max_hp, :name]
iex> Map.values(pc1)
[Player, 1000, "kminwoog"]
iex(13)> %{__struct__: Player, max_hp: 1000, name: "kminwoog"} == pc1
true
iex> %Player{} == Player.__struct__()
true
```




