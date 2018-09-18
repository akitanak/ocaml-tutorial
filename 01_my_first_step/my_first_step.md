
# はじめの一歩
## コメント

- 1行コメント
```
(* this is 1 line comment *)
```

- 複数行に渡るコメント
```
(* this is
 * multiline
 * comment
 *)
```

- コメントはネストできる。
```
(*
  (* comment can nest in OCaml *)
*)
```

## 関数呼び出し

- 多くの言語では関数を呼び出す際に引数は括弧の中にカンマ区切りで指定するが、OCamlは括弧を付けないし、カンマも付けない。
- 下記のように記述すると `get_string_from_user` の戻り値を `repeated` の第1引数として渡すことになる。

  ```ocaml
  repeated (get_string_from_user "Please type in a string.") ```

## 関数定義

- OCaml は強い静的型付けされた言語
- 型推論によって型を抽出するので明示する必要はない。
- 暗黙の型変換は無い
- 演算子の多重定義は禁止されている
  - そのため、整数と浮動小数点数では加算の演算子も異なる(整数: `+` , 浮動小数点数: `+.`)
- OCaml は関数の最後の式を返値とする

浮動小数点数を引数に2つとり、平均を求める関数は次の様に定義できる。
```ocaml
let average a b =
  (a +. b) /. 2.0;;
```

## 基本の型

- int: 32bit cpu では31bit符号付き整数、64bit cpu では63bit符号付き整数
- float: IEEE 倍精度浮動小数点数、C の double と同じ
- bool: true/falseとなる真偽値
- char: 8bit文字
- string: 文字列
- unit: () と書くもの

32bit整数型を使いたいときは `nativeint` 型を使う。

## 明示的/暗黙の型変換

- OCaml では暗黙の型変換は一切行われない。
- 次のような式はエラーになる
  ```ocaml
  1 + 2.5;;
  1 +. 2.5;;
  ```
- もし整数と浮動小数点数を足し算したいなら、明示的なキャストが必要。次のように書ける。
  ```ocaml
  (float_of_int 1) +. 2.5;;
  ```
- float_of_intはよく使うので略記として `float` と書けば良い。
  ```ocaml
  float 1 +. 2.5;;
  ```

## 普通の関数と再帰関数

- 単に `let` ではなく `let rec` と明示的に書かないと再帰的には使えない。
- 再帰関数の例
  ```ocaml
  let rec range a b =
    if a > b then []
    else a :: range (a+1) b;;
  ```
- `let` と `let rec` の違いは関数名のスコープのみで、 `let` で定義されていた場合は関数内の `range` の呼び出しのところで既に定義済みの `range` を探す。
- `let` と `let rec` には性能的な違いはない。

## 関数の型

OCaml は 引数 `arg1`, `arg2`, ..., `argn`を取り、`rettype`型を返すとき、
コンパイラは次のように表示する
```
f : arg1 -> arg2 -> ... -> argn -> rettype
```

関数 `repeated` は文字列と数値を引数にとって文字列を返すので、次のようになる。
```
repeated : string -> int -> string
```

### 多相関数

引数が何でも良いような関数。
例えば、引数を受け取るが、それを無視して常に3を返す関数を考える。
```ocaml
let give_me_a_three x = 3
```
このような関数は、コンパイルされると次にように表記される。
```
give_me_a_three : 'a -> int
```
`'a` はどのような型でも良いことを意味する。

## 型推論

OCaml は関数や変数の型を明示的に宣言する必要がなく、 OCaml が代わりに型を推論してくれる。

例
```ocaml
# let average a b =
    (a +. b) /. 2.0;;
val average : float -> float -> float = <fun>
```

`+.` は常に2つの `float` を引数にとる関数であるため、`a`, `b` は `float` になる。`/.` も `float` を返す関数であり、これが `average` の戻り値になるから `average` の戻り値も `float` になると推論される。
