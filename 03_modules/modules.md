# モジュール

## 基本的な使い方

- OCamlではあらゆるコードがモジュールにくるまれている。
- モジュール自身は別のモジュールのサブモジュールになれるが、あまりやらない。
- `amodule.ml`, `bmodule.ml` という2つのファイルを使うとすると、これらは`Amodule`, `Bmodule` という名前のモジュールとなる。
- モジュールに定義された関数は `{module_name}.{value/type/constractor}` で呼び出す。

例

`amodule.ml` の中に次のコードがあったとする。
```ocaml
let hello () = print_endline "Hello"
```

これを別のモジュールで使う場合、次のようになる。
```ocaml
Amodule.hello ()
```

また、次のように書けば、モジュールの中身に直接アクセスできる。
```ocaml
open Amodule;;
hello ();;
```

醜い `;;` を避けたいならば、次のように書ける。
```ocaml
open Amodule
let () =
  hello()
```

## インタフェースとシグネチャ

モジュールに関数、型、サブモジュールを定義すると基本的に全て外からアクセス可能になる。

補助関数、補助型などの外部に公開したくない場合、モジュールインタフェースを定義することで、必要なものだけ公開することができる。モジュールインタフェースやシグネチャは`.mli`ファイルに定義する。

例えば、次のような`amodule.ml`があったとする。
```ocaml
let message = "Hello"
let hello () = print_endline message;;
``
このモジュールの`hello` だけ公開するのであれば、
`amodule.mli` に次のように定義する。
```ocaml
val hello : unit -> unit
```
このとき、ocamldoc がサポートしているフォーマットに基づいて`.mli`ファイルにドキュメントを残すとよい。

`.mli`ファイルはマッチする`.ml`ファイルの直前にコンパイルされなければならない。

## 抽象型

モジュールではしばしば新しい型定義をする。
次のような日付を表す簡単なレコード型があったとする。
```ocaml
type date = { day : int; month : int; year : int }
```
これを `.mli`ファイルに書き出す時に４つの選択肢がある。
1. 型はシグネチャから完全に省略される
2. 型定義をシグネチャにコピー＆ペーストする
3. 型を抽象化する (名前だけを与える)
4. レコードのフィールドを読み出し専用にする (`type data = private { ... }`)

3番目の場合なら次のようになる。
```ocaml
type date
```
これだと、モジュールのユーザーは型`date`のオブジェクトは扱えるが、レコードのフィールドにはアクセスできない。
モジュールで提供される関数を使って日付を生成、日付の差の計算、日付を年換算する関数を提供するとすると次のようになる。
```ocaml
type date
val create : ?date:int -> ?months:int -> ?years:int -> unit -> date
val sub : date -> date -> date
val years : date -> float
```

## サブモジュール
### サブモジュールの実装

モジュールの中にサブモジュールを定義することができる。
サブモジュールは次のように定義できる。 `example.ml`ファイルがあるとすると次のようになる。
```ocaml
module Hello = struct
  let message = "Hello"
  let hello () = print_endline message
end
let goodbye () = print_endline "Goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
```

別のファイルから参照するには次のように書く。
```ocaml
let () =
  Example.Hello.hello ();
  Example.goodbye ()
```

### サブモジュールのインタフェース
サブモジュールのインタフェースも制限できる。これはモジュール型と呼ばれる。
さきほどの `exa,ple.ml` でやってみると次のようになる。
```ocaml
module Hello : sig
  val hello : unit -> unit
end =
struct
  let message = "Hello"
  let hello () = print_endline message
end

let goodbye () = print_endline "Goodbye"
let hello_goodbye () =
  Hello.hello ();
  goodbye ()
```
コードブロック一つに全部書くのはエレガントではないので、普通はモジュールとシグネチャを分割定義する。
```ocaml
module type Hello = sig
  val hello : unit -> unit
end

module Hello : Hello_type = struct
  ...
end
```
`Hello_type` は名前つきモジュール型であり、他のモジュールインタフェース定義に再利用できる。この実用性はファンクタで明らかになる。

## ファンクタ

ファンクタはおそらくOCamlの中でもっとも複雑な特徴のひとつ。

### ファンクタとはなにか？なぜ必要か？

ファンクタは別のモジュールでパラメータ化されるモジュールであり、関数が別の値(引数)によってパラメータ化された値であるのと同じようなもの。

ファンクタは値で型をパラメータ化できる。
例えば、 `int n` を引数にとって、長さ `n` の配列だけ排他的に動作するような配列操作を集めたものを返すファンクタを定義できる。







