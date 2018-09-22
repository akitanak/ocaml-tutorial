# OCaml プログラムの構造

## ローカルな"変数" (本当はローカルな式)

`average` 関数にローカルな変数を加えてみると
```ocaml
# let average a b =
    let sum = a +. b in
    sum /. 2.0;;
val average : float -> float -> float = <fun>
```
標準の句 `let name = expression in` を使って、ローカルな式に名前を付けて定義する。これで `name` は後に関数の中で、`expression` の代わりに使えるようになる。
式の終わりの `;;` でコードのブロックが閉じる。in のあとでインデントはしない。
`let ... in` は人るの文であると、考える。
OCaml　においては、上記例の `sum` は `a +. b` の略名にすぎない。`sum` には代入できないし、その値を変えることもできない。

## グローバルな"変数" (本当はグローバルな式)

例
```ocaml
let html =
  let content = read_whole_file file in
  GHtml.html_from_string content
  ;;

let menu_bold () =
  match bold_button#active with
  | true -> html#set_font_style ~enable:[`BOLD] ()
  | false -> html#set_font_style ~disable:[`BOLD] ()
  ;;

let main () =
  (* code omitted *)
  factory#add_item "Cut" ~key:_X ~callback: html#cut
  ;;
```

グローバルな名前の定義は、ローカルな"変数"と同じく、変数ではまるでなく式の略名に過ぎない。
そのため、上の例では `html` は `html` ポインタを格納していないし、
`html` への代入もできない。

## Let 束縛

`let ...` を使う時、 **let 束縛** をすると呼ぶ事がある。

## 参照: 本当の変数

本当の変数が必要な時、 **参照** を使う。
これは C/C++のポインタと似ている。

`int`への参照を作るには次にようにする。
```ocaml
# let my_ref = ref 0;;
val my_ref : int ref = {contents = 0}
```

代入するには次のようにする。
```ocaml
# my_ref := 100;;
- : unit = ()
```

参照の中身を取り出すには次のようにする。
```ocaml
# !my_ref;;
- : int = 100
```

参照はそう頻繁には使わず、 `let name = expression in` を使って関数内のローカルな式に名前をつけることの方が多い。

## 入れ子関数

入れ子関数とは、関数の中で定義された関数である。
入れ子関数の名前は定義されたブロック内で有効である。
入れ子関数からはそれを含む関数の変数のうち、入れ子関数が定義される時点で見えているものはすべて参照可能である。これをレキシカル・スコーピングと呼ぶ。
例
```ocaml
# let read_whole_channel chan =
    let buf = Buffer.create 4096 in
    let rec loop () =
      let newline = input_line chan in
      Buffer.add_string buf newline;
      Buffer.add_char buf '\n';
      loop()
    in
    try
      loop()
    with
      End_of_file -> Buffer.contents buf;;
val read_whole_channel : in_channel -> string = <fun>
```

ローカルな入れ子関数は次のように定義する。
`let name arguments = function-definition in`

上記例は、入れ子関数の中で自身を呼ぶ再帰関数となっており、その場合は前述の通り `let rec` を使う。

## モジュールと `open`

OCaml には様々なモジュールがたくさん用意されている。
例えばUnixなら `/usr/lib/ocaml/` にモジュールが置かれている。
これらのモジュールを使うには、プログラムの最初に `open module-name;;` 宣言するか、頭にモジュール名を足して呼べばよい。その場合は、 `module-name.func` となる。

## Pervasives モジュール

`Pervasives` モジュールは唯一 `open` する必要がないモジュール。
組み込み型の基本操作を提供する。

## モジュールをリネームする

あるモジュールをまるごとインポートするのは嫌だが、モジュール名をいちいち全てタイプするのが面倒な場合、モジュールをリネームする事ができる。

```ocaml
module Gr = Graphics;;

Gr.open_graph " 640x480";;
Gr.fill_circle 320 240 240;;
read_line();;
```

## ;;と;、使ったり、削ったり

### ;; について
- `;;` を使うべき時は、コードのトップレベルにある文を区切る時。
- `;;` を削っても良い時は、
  - `let`キーワードの前
  - `open`キーワードの前
  - `type`キーワードの前
  - ファイルの最後の最後

### ; について
- セミコロン一つの`;`は、シーケンスポイントということになっている。
- `let ... in` は文と思って、その後ろに一重の `;` は置かない。
- 他の全ての文で、コードのブロック内にあるものには一重の`;`をつける。ただし、一番最後のは除く

