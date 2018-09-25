# The module system

この章では OCaml のモジュールシステムについて紹介する。

## 2.1 Structures

モジュールの主な動機はデータ型の定義とその型に関連する操作を一緒にパッケージングして、これらの定義に一貫した命名規則を適用することである。

そのようなパッケージは構造体と呼ばれ、`struct ... end` で導入される。この構造体には任意の一連の定義が含まれる。
通常、構造体は `module` によって名前を指定される。

次の例は一つの構造体にpriority queue型とそれに対する操作を一緒にパッケージングしている。
```ocaml
# module PrioQueue =
  struct
    type priority = int
    type 'a queue = Empty | Node of priority * 'a * 'a queue * 'a queue
    let empty = Empty
    let rec insert queue prio elt =
      match queue with
        Empty -> Node(prio, elt, Empty, Empty)
      | Node(p, e, left, right) ->
        if prio <= p
        then Node(prio, elt, insert right p e, left)
        else Node(p, e, insert right prio elt, left)
    exception Queue_is_empty
    let rec remove_top = function
        Empty -> raise Queue_is_empty
      | Node(prio, elt, left, Empty) -> left
      | Node(prio, elt, Empty, right) -> right
      | Node(prio, elt, (Node(lprio, lelt, _, _) as left), (Node(rprio, relt, _, _) as right)) ->
        if lprio <= rprio
        then Node(lprio, lelt, remove_top left, right)
        else Node(rprio, relt, left, remove_top right)
    let extract = function
        Empty -> raise Queue_is_empty
      | Node(prio, elt, _, _) as queue -> (prio, elt, remove_top queue)
  end;;
module PrioQueue :
  sig
    type priority = int
    type 'a queue = Empty | Node of priority * 'a * 'a queue * 'a queue
    val empty : 'a queue
    val insert : 'a queue -> priority -> 'a -> 'a queue
    exception Queue_is_empty
    val remove_top : 'a queue -> 'a queue
    val extract : 'a queue -> priority * 'a * 'a queue
  end
```

構造体の外からはドット記法で参照できる。
```ocaml
# PrioQueue.insert PrioQueue.empty 1 "hello";;
- : string PrioQueue.queue =
PrioQueue.Node (1, "hello", PrioQueue.Empty, PrioQueue.Empty)
```

openを使って、スコープにモジュールの中身を展開することもできる。
```ocaml
# open PrioQueue;;
# insert empty 1 "hello";;
- : string PrioQueue.queue = Node (1, "hello", Empty, Empty)
```


## 2.2 Signatures

## 2.3 Functors

## 2.4 Functors and type abstraction

## 2.5 Modules and separate compilation
