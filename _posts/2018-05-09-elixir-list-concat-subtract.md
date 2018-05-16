---
title:  "List concatenation and subtraction in elixir"
categories: 
  - elixir
last_modified_at: 2018-05-09T11:54:59-09:00
tags: elixir elixirlang TIL
comments: true
---

엘릭서에서 리스트을 연결하거나 뺄 때 `Kernel.++/2`, `Kernel.--/2`를 가끔 사용한다.

두 리스트의 A = [1,2,3], B = [7,8,9] 를 연결하는 `A ++ B` 를 작성하면 실제로는 다음과 같이 결과가 출력된다.

```elixir
iex> [1,2,3] ++ [7,8,9]
[1,2,3,7,8,9]
```

여기서 `순서가 중요하지 않고` 리스트에 `하나`의 element를 추가할 때는 `|` 연산자를 사용해서 앞부분에 연결할 수 있다는 것에 주목할 필요가 있다.

엘릭서에서 리스트는 singly linked list이기 때문에 기존에 존재하는 리스트의 뒷부분에 추가하는 것보다 앞부분에 추가하는 것이 비용이 싸다.

```elixir
iex> [1 | [2,3,4]]
[1,2,3,4]
```

하나의 엘리먼트가 아닌 여러 엘리먼트로 확장하면 두 리스트의 concatenation이 된다.

```elixir
iex> [1,2,3 | [7,8,9]]
[1,2,3,7,8,9]
```

엘릭서 코드로 List concatenation 함수를 작성하면 다음과 같은 코드와 같은 형태이다.

```elixir
defmodule List2 do
  def concat([], b), do: b
  def concat([h|t], b), do: [h | concat(t, b)]
end

iex> List2.concat([1,2,3], [7,8,9])
[1,2,3,7,8,9]
```


### Common pitfalls

#### Improper list

`|` 연산자의 오른쪽 값이 리스트가 아닌 값일 때, `부적당한 리스트`가 된다.  
`리스트(?)이긴 하다`  
하지만 대부분의 상황에서 리스트로서의 역할은 하지 못한다. Enumable하지 않다.

```elixir
iex> l = [1,2,3 | 7]
[1,2,3 | 7]
iex> is_list(l)
true
```

#### ++, \-- operator associativity

결론 부터 말하면 연산자의 결합 순서는 `right to left (right associative)` 이다.

```elixir
iex> [1,2,3] -- [1,2,3] -- [3]
[3]
# 왼쪽에서 부터 연산하는 것이 아니라 오른쪽(뒤)에서 부터 연산의 우선순위를 가진다.
```

언어마다 연산자간의 우선순위와 결합 순서(associativity)는 조금씩 다르다.  
여러 언어를 사용하다 보면 그것들을 일일이 인지하면서 사용하기 힘들다.  
그래서 나는 좀더 명확하고 혼란스럽지 않도록 아래와 같은 코드로 풀어쓰는 형태를 선호한다.

```elixir
iex> list = [1,2,3] -- [3]
iex> [1,2,3] -- list
[3]
```
