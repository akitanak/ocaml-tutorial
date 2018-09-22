# The Core Language
## 1.1 Basics

OCamlのREPLではOCamlの文をかいて`;;`で終了すると、コンパイル、実行され、結果が返ってくる。

```ocaml
# 1 + 2 * 3;;
- : int = 7
```
```ocaml
# let pi = 4.0 *. atan 1.0;;
val pi : float = 3.14159265358979312
```
```ocaml
# let square x = x *. x;;
val square : float -> float = <fun>
```
```ocaml
# square (sin pi) +. square (cos pi);;
- : float = 1.
```

OCamlは各文の値と型を計算する。
関数のパラメータも、関数内での利用方法から型を推論するので明示的に指定する必要がない。
floatとintegerは型が区別されるだけでなく、演算子も区別される。
integerなら `+`、`*`、floatなら`+.`、`*.`。

再帰関数は `let rec` を使って定義する。
```ocaml
# let rec fib n =
  if n < 2 then n else fib (n-1) + fib (n-2);;
val fib : int -> int = <fun>
# fib 10;;
- : int = 55
# fib 0;;
- : int = 0
# fib 1;;
- : int = 1
```

## 1.2 Data types

OCamlの基本データ型には次のものがある。
- int
- float
- boolean
- char
- string

```ocaml
(1 < 2) = false;;
- : bool = false
# 'a';;
- : char = 'a'
# 'Hello world';;
Error: Syntax error
# "Hello world";;
- : string = "Hello world"
```
char はシングルクォート、 string はダブルクォートでくくる。

定義済みデータ構造は次のものがある。
- tuple
- array
- list

自分でデータ構造を定義するための仕組みもある。

### List

定義する際は、`[]` で囲って、要素を `;` で区切る。
Listの先頭に要素を追加するためには `::` (cons) を使う。

```ocaml
# let l = ["is"; "a"; "tale"; "told"; "etc."];;
val l : string list = ["is"; "a"; "tale"; "told"; "etc."]
# "Life" :: l;;
- : string list = ["Life"; "is"; "a"; "tale"; "told"; "etc."]
```

当然、listの要素の型は揃える必要がある。
```ocaml
# let m = ["is"; 'a'; "tale"; "told"; "etc."];;
Error: This expression has type char but an expression was expected of type
         string
```

他のOCamlのデータ構造と同様に list は明示的にメモリーの確保・開放する必要はない。
OCamlではメモリーの管理は自動的に行われる。
明示的にポインタを扱うこともない。

OCamlの多くのデータ構造では、パターンマッチで中身を調査、展開できる。
list のパターンマッチは listの志木と同じ表現で指定できる。

```ocaml
# let rec sort lst =
    match lst with
      [] -> []
    | head :: tail -> insert head (sort tail)
  and insert elt lst =
    match lst with
      [] -> [elt]
    | head :: tail -> if elt <= head then elt :: lst else head :: insert elt tail
  ;;
val sort : 'a list -> 'a list = <fun>
val insert : 'a -> 'a list -> 'a list = <fun>
```

```ocaml
# sort l;;
- : string list = ["a"; "etc."; "is"; "tale"; "told"]
```
```ocaml
# let m = ["b"; "c"; "d"];;
val m : string list = ["b"; "c"; "d"]
# insert "g" m;;
- : string list = ["b"; "c"; "d"; "g"]
# insert "a" m;;
- : string list = ["a"; "b"; "c"; "d"]
```

`sort` の型推論結果 `'a list -> 'a list` は、`sort` は任意の型のリストを引数にとり、
同じの型のリストを返すことを意味している。
`'a` は型変数である。
比較演算子は OCaml ではポリモーフィックなので、`sort` は任意の型のリストに適用することができる。

```ocaml
# sort [6;2;5;3];;
- : int list = [2; 3; 5; 6]
```
```ocaml
# sort [3.14;2.718];;
- : float list = [2.718; 3.14]
```
`sort` は入力の list を変更せず、昇順にソートされた新しい list を作って返す。
OCaml では一度作った list を変更する方法はない。イミュータブルなのである。
多くの OCaml のデータ構造はイミュータブルだが、2、3つはミュータブルなものもある。

Ocaml の複数の引数を取る関数の型の記法は次のようになる。
`arg1_type -> arg2_type -> ... -> return_type`
例えば、`insert` は `'a -> 'a list -> 'a list` と定義されるが、
これは任意の型の値とその型を要素に持つ list を引数に取り、入力と同じ型を持つ list を返すことを意味する。

## 1.3 Functions as Values

OCaml は関数型言語であり、関数を他のデータと同様に自由に渡すことができる。

```ocaml
# let deriv f dx = function x -> (f (x +. dx) -. f x) /. dx;;
val deriv : (float -> float) -> float -> float -> float = <fun>
```
```ocaml
# let sin' = deriv sin 1e-6;;
val sin' : float -> float = <fun>
```
```ocaml
# sin' pi;;
- : float = -1.00000000013961143
```

なので、関数合成できる。

```ocaml
# let compose f g = function x -> f(g x);;
val compose : ('a -> 'b) -> ('c -> 'a) -> 'c -> 'b = <fun>
```
```ocaml
# let cos2 = compose square cos;;
val cos2 : float -> float = <fun>
# cos2 pi;;
- : float = 1.
```

関数を引数にとれる関数のことを「ファンクショナルな関数」、「高階関数」と呼ぶ。
高階関数はイテレータやデータ構造をまたいだジェネリックなオペレーションを生成するのに便利。

OCaml ライブラリの `List.map` は list の各要素に与えられた関数を適用し、その結果を返す。
```ocaml
# List.map (function n -> n * 2 + 1) [0;1;2;3;4];;
- : int list = [1; 3; 5; 7; 9]
```

この `map` は次のように定義できる。
```ocaml
# let rec map f l =
    match l with
      [] -> []
    | hd :: tl -> f hd :: map f tl;;
val map : ('a -> 'b) -> 'a list -> 'b list = <fun>
```
```ocaml
# map (function n -> n * 2 + 1) [0;1;2;3;4];;
- : int list = [1; 3; 5; 7; 9]
```

## 1.4 Records and Variants

ユーザー定義データ構造にはレコードと変数？がある。
どちらも `type` をつかって定義できる。

以下は分数を型で定義した例である。
```ocaml
# type ratio = {num: int; denom: int};;
type ratio = { num : int; denom : int; }
```
```ocaml
# let add_ratio r1 r2 =
    {num = r1.num * r2.denom + r2.num * r1.denom;
      denom = r1.denom * r2.denom};;
val add_ratio : ratio -> ratio -> ratio = <fun>
```
```ocaml
# add_ratio {num=1; denom=3} {num=2; denom=5};;
- : ratio = {num = 11; denom = 15}
```

レコードのフィールドにはパターンマッチでアクセスできる。
```ocaml
# let integer_part r =
    match r with
      {num=num; denom=denom} -> num / denom;;
val integer_part : ratio -> int = <fun>
```

パターンマッチに1ケースしかない場合であれば、
引数 `r` をレコードのパターンで直接展開しても安全である。
```ocaml
# let integer_part {num=num; denom=denom} = num / denom;;
val integer_part : ratio -> int = <fun>
```

不要なフィールドは省略することもできる。
```ocaml
# let get_denom {denom=denom} = denom;;
val get_denom : ratio -> int = <fun>
```

オプションで、フィールドが欠落していることをフィールドのリストの末尾をワイルドカード `_` で終えることで明示することができる。

```ocaml
# let get_num {num=num; _ } = num;;
val get_num : ratio -> int = <fun>
```

レコードを生成する際に次のようにフィールドを略記することもできる。
```ocaml
# let ratio num denom = {num; denom};;
val ratio : int -> int -> ratio = <fun>
```

レコードのいくつかのフィールドを同時に更新することもできる。
```ocaml
# let integer_product integer ratio = { ratio with num = integer * ratio.num };;
val integer_product : int -> ratio -> ratio = <fun>
```
このようなファンクショナルな更新記法を用いると、`with` より右手の更新されるフィールド以外は左手のレコードからコピーされる。

variant型の宣言はそのvariant型で値が取り得る全ての型を列挙する。
それぞれのケースは名前によって区別され、コンストラクタと呼ばれ、
variant型の値の生成とパターンマッチによる検査を提供する。
コンストラクタの名前は変数名と区別するために先頭大文字にされる。

次の例は、整数と浮動小数点数をミックスしたvariant型である。
```ocaml
# type number = Int of int | Float of float | Error;;
type number = Int of int | Float of float | Error
```

この宣言は`number` 型は整数か浮動小数点数、または不正な操作結果を意味する定数 `Error` を表現している。

取り得る定数を示した列挙型は variant型の特別なケースである。
```ocaml
# type sign = Positive | Negative;;
type sign = Positive | Negative
```
```ocaml
# let sign_int n = if n >= 0 then Positive else Negative;;
val sign_int : int -> sign = <fun>
# sign_int 1;;
- : sign = Positive
# sign_int (-1);;
- : sign = Negative
```

`number` 型に対する算術的な演算を定義するために、2つの数にパターンマッチを使うと次のようになる。
```ocaml
# let add_num n1 n2 =
    match (n1, n2) with
      (Int i1, Int i2) ->
        (* Check for overflow of integer addition *)
        if sign_int i1 = sign_int i2 && sign_int (i1 + i2) <> sign_int i1
        then Float(float i1 +. float i2)
        else Int(i1 + i2)
    | (Int i1, Float f2) -> Float(float i1 +. f2)
    | (Float f1, Int i2) -> Float(f1 +. float i2)
    | (Float f1, Float f2) -> Float(f1 +. f2)
    | (Error, _) -> Error
    | (_, Error) -> Error;;
val add_num : number -> number -> number = <fun>
```

他の variant型についての興味深い例は組み込みの型 `'a` の値か値が無いことを表す `'a option` 型である。
```ocaml
# type 'a option = Some of 'a | None;;
type 'a option = Some of 'a | None
```

この型は一般的なシチュエーションで失敗する可能性のある関数を定義するときにとても役立つ。
```ocaml
# let safe_square_root x = if x > 0. then Some(sqrt x) else None;;
val safe_square_root : float -> float option = <fun>
```
```ocaml
# safe_square_root 2.;;
- : float option = Some 1.41421356237309515
# safe_square_root (-2.);;
- : float option = None
```

最も一般的な variant 型の使い方は再帰的なデータ型の表現である。
例えば、バイナリツリー型を考えてみる。
```ocaml
# type 'a btree = Empty | Node of 'a * 'a btree * 'a btree;;
type 'a btree = Empty | Node of 'a * 'a btree * 'a btree
```

この定義は、任意の型 `'a` を持つバイナリツリーは空か、型 `'a` の１つの値と2つの型 `'a` の値を持つサブツリーをもつノードを持つと定義される。

バイナリツリーの操作は再帰関数で表現される。
次は整列されたバイナリツリーをルックアップし挿入する操作を行う例である。
```ocaml
# let rec member x btree =
    match btree with
      Empty -> false
    | Node(y, left, right) ->
      if x = y then true else
      if x < y then member x left else member x right;;
val member : 'a -> 'a btree -> bool = <fun>
```
```ocaml
# let rec insert x btree =
    match btree with
      Empty -> Node(x, Empty, Empty)
    | Node(y, left, right) ->
      if x <= y then Node(y, insert x left, right)
        else Node(y, left, insert x right);;
val insert : 'a -> 'a btree -> 'a btree = <fun>
```

