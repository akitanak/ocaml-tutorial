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

## 1.5 Imperative features

ここまでの例は純粋なアプリカティブ・スタイルで書かれていたが、 OCaml は命令的な機能も備えている。それにはよくある `while` と `for` ループ、 配列のようなミュータブルなデータストラクチャも含まれている。配列は `[|` と `|]` の間に `;` で区切って要素を並べることで生成できる。もしくは、 `Array.make` で生成し、あとで、代入により値を埋めることができる。

例えば、次のように２つのベクターの和を求めることができる。
```ocaml
# let add_vect v1 v2 =
  let len = min (Array.length v1) (Array.length v2) in
  let res = Array.make len 0.0 in
  for i = 0 to len - 1 do
    res.(i) <- v1.(i) +. v2.(i)
  done;
  res;;
val add_vect : float array -> float array -> float array = <fun>
```
```ocaml
# add_vect [|1.;0.3;1.5; 4.|] [|0.5; 3.5; 3.|]
  ;;
- : float array = [|1.5; 3.8; 4.5|]

レコードのフィールドは、 レコード型を定義する際に `mutable` で宣言されていれば、代入によって変更することができる。
```ocaml
# type mutable_point = { mutable x: float; mutable y: float };;
type mutable_point = { mutable x : float; mutable y : float; }
```
```ocaml
# let translate p dx dy =
  p.x <- p.x +. dx; p.y <- p.y +. dy;;
val translate : mutable_point -> float -> float -> unit = <fun>
```
```ocaml
# let mypoint = { x = 0.0; y = 0.0 };;
val mypoint : mutable_point = {x = 0.; y = 0.}

# translate mypoint 1.0 2.0;;
- : unit = ()

# mypoint;;
- : mutable_point = {x = 1.; y = 2.}
```

OCaml は変数の概念 (代入によって現在の値を変更する) がないが、標準ライブラリは `!` 演算子を使って参照先の現在の値を取得することができ、 `:=` で値を代入できるミュータブルな間接参照を提供する。

```ocaml
# let insertion_sort a =
  for i = 1 to Array.length a - 1 do
    let val_i = a.(i) in
    let j = ref i in
    while !j > 0 && val_i < a.(!j -1) do
      a.(!j) <- a.(!j-1);
      j := !j - 1
    done;
    a.(!j) <- val_i
  done;;
val insertion_sort : 'a array -> unit = <fun>
```
```ocaml
# let a = [| 5; 3; 6; 2; 9; 1; 0 |];;
val a : int array = [|5; 3; 6; 2; 9; 1; 0|]

# insertion_sort a;;
- : unit = ()
# a;;
- : int array = [|0; 1; 2; 3; 5; 6; 9|]
```

参照は２回の関数呼び出しの間で現在の状態を維持する関数を書くのに役立つ。
次の例の疑似乱数のジェネレータは最後に返した値を参照で保持する。
```ocaml
# let current_rand = ref 0;;
val current_rand : int ref = {contents = 0}

# let random () =
  current_rand := !current_rand * 25713 + 1345;
  !current_rand;;
val random : unit -> int = <fun>
```
```ocaml
# random ();;
- : int = 1345

# !current_rand;;
- : int = 1345
```

参照はミュータブルなフィールド1つだけをもつレコードとして実装されている。
```ocaml
# type 'a ref = { mutable contents: 'a };;
type 'a ref = { mutable contents : 'a; }

# let (!) r = r.contents;;
val ( ! ) : 'a ref -> 'a = <fun>

#  let (:=) r newval = r.contents <- newval;;
val ( := ) : 'a ref -> 'a -> unit = <fun>
```

特別なケースで、ポリモーフィズムを維持したままデータ構造の中にポリモーフィックな関数を格納する必要があるとする。これをするには、ポリモーフィズムはグローバルな定義に対してのみ自動的に導入されるためユーザーによる型アノテーションが必要である。
レコードのフィールドにポリモーフィック型を明示することができる。
```ocaml
# type idref = { mutable id: 'a. 'a -> 'a };;
type idref = { mutable id : 'a. 'a -> 'a; }
```
```ocaml
# let r = {id = fun x -> x };;
val r : idref = {id = <fun>}

# let g s = (s.id 1, s.id true);;
val g : idref -> int * bool = <fun>

# r.id <- (fun x -> print_string "called id\n"; x);;
- : unit = ()

# g r;;
called id
called id
- : int * bool = (1, true)
```

## 1.6 Exceptions

例外的な状態を知らせ、ハンドリングするために OCaml は例外を提供する。
例外は `exception` を使って宣言し、 `raise` 演算子で投げる。

例
```ocaml
# exception Empty_list;;
exception Empty_list
```
```ocaml
# let head l =
    match l with
      [] -> raise Empty_list
    | hd :: tl -> hd;;
val head : 'a list -> 'a = <fun>

# head [1; 2];;
- : int = 1

# head [];;
Exception: Empty_list.
```

例外は、標準ライブラリでも、正常に関数を完了することができないようなケースを知らせるのに使われている。例えば、与えられたキーに関連するデータをペアのリストの中から返す `List.assoc` 関数では、リストの中にキーが出てこなかった場合に `Not_found` 例外を投げる。
```ocaml
# List.assoc 1 [(0, "zero"); (1, "one")];;
- : string = "one"

# List.assoc 2 [(0, "zero"); (1, "one")];;
Exception: Not_found.
```

例外は `try...with` を使って捕捉することができる。
```ocaml
# let name_of_binary_digit digit =
  try
    List.assoc digit [0, "zero"; 1, "one"]
  with Not_found ->
    "not a binary digit";;
val name_of_binary_digit : int -> string = <fun>
```
```ocaml
# name_of_binary_digit 0;;
- : string = "zero"

# name_of_binary_digit (-1);;
- : string = "not a binary digit"
```

`with` の部分は `match` と同じシンタックスと動作で例外の値をパターンマッチングできる。なので、いくつかの例外を１つの `try...with` で捕捉することができる。

```ocaml
# let temporarily_set_reference ref newval funct =
  let oldval = !ref in
  try
    ref := newval;
    let res = funct () in
    ref := oldval;
    res
  with x ->
    ref := oldval;
    raise x;;
val temporarily_set_reference : 'a ref -> 'a -> (unit -> 'b) -> 'b = <fun>
```

## 1.7 Symbolic processing of expressions

シンボリックプロセッシング?のための典型的な OCaml の使用例を紹介する。
次の variant 型は、これから行おうとしている操作の式を表現している。
```ocaml
# type expression =
    Const of float
  | Var of string
  | Sum of expression * expression (* e1 + e2 *)
  | Diff of expression * expression (* e1 - e2 *)
  | Prod of expression * expression (* e1 * e2 *)
  | Quot of expression * expression (* e1 / e2 *)
  ;;
type expression =
    Const of float
  | Var of string
  | Sum of expression * expression
  | Diff of expression * expression
  | Prod of expression * expression
  | Quot of expression * expression
```
はじめに、与えられた変数名と値のマップで式を評価する関数を定義する。

```ocaml
# exception Unbound_variable_of_string;;
exception Unbound_variable_of_string

# let rec eval env exp =
  match exp with
    Const c -> c
  | Var v ->
    (try List.assoc v env with Not_found -> raise (Unbound_variable v))
  | Sum(f, g) -> eval env f +. eval env g
  | Diff(f, g) -> eval env f -. eval env g
  | Prod(f, g) -> eval env f *. eval env g
  | Quot(f, g) -> eval env f /. eval env g;;
val eval : (string * float) list -> expression -> float = <fun>
```
```ocaml
# eval [("x", 1.0); ("y", 3.14)] (Prod(Sum(Var "x", Const 2.0), Var "y"));;
- : float = 9.42
```

変数 `dv` に関する式の微分を定義する。
```ocaml
let rec deriv exp dv =
match exp with
  Const c -> Const 0.0
| Var v -> if v = dv then Const 1.0 else Const 0.0
| Sum(f, g) -> Sum(deriv f dv, deriv g dv)
| Diff(f, g) -> Diff(deriv f dv, deriv g dv)
| Prod(f, g) -> Sum(Prod(f, deriv g dv), Prod(deriv f dv, g))
| Quot(f, g) -> Quot(Diff(Prod(deriv f dv, g), Prod(f, deriv g dv)), Prod(g, g))
;;
```
```ocaml
# deriv (Quot(Const 1.0, Var "x")) "x";;
- : expression =
Quot (Diff (Prod (Const 0., Var "x"), Prod (Const 1., Const 1.)),
 Prod (Var "x", Var "x"))
```

