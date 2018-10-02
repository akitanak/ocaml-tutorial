# Objects in OCaml

この章は OCaml のオブジェクト指向のフィーチャの概要を説明する。

OCaml におけるオブジェクトとクラス、型の関係は Java や　C++ のような一般的なオブジェクト指向言語とは異なる。なので、同じキーワードがあっても同じものだと思わないほうがよい。
オブジェクト指向なフィーチャーはそれらの言語よりも OCaml では使われない。
OCaml は modules や functors のようなより適切な場合が多い代替手段をもっており、実際、多くのOCamlプログラムでは、オブジェクトを全く使っていない。

## 3.1 Classes and objects

次の `point` クラスは、１つのインスタンス変数 `x` と 2つのメソッド `get_x` と `move` を定義している。インスタンス変数の初期値は `0` である。変数 `x` はミュータブルで宣言されており、 `move` メソッドはその値を変更できる。

```ocaml
# class point =
  object
    val mutable x = 0
    method get_x = x
    method move d = x <- x + d
  end;;
class point :
  object val mutable x : int method get_x : int method move: int -> unit end
```

`point` クラスのインスタンスとして、 `p` を生成する。

```ocaml
# let p = new point;;
val p : point = <obj>
```
`p` の型は `point` であることに注意すること。
これは、上記のクラス定義によって自動的に定義される生薬系です。
これはオブジェクトタイプ<get_x: int, move: int -> unit>を表し、クラス `point` のメソッドとその型をリストする。

`p` のメソッドを実行してみる。

```ocaml
# p#get_x;;
- : int = 0

# p#move 3;;
- : unit = ()

# p#get_x;;
- : int = 3
```

クラス本体の評価は、オブジェクト生成時にのみ行われる。そのため、次の例ではインスタンス変数 `x` は２つの異なるオブジェクトでは異なる値に初期化される。

```ocaml
# let x0 = ref 0;;
val x0 : int ref = {contents = 0}
```
```ocaml
# class point =
  object
    val mutable x = incr x0; !x0
    method get_x = x
    method move d = x <- x + d
  end;;
class point :
  object val mutable x : int method get_x : int method move : int -> unit end
```
```ocaml
# new point#get_x;;
- : int = 1

# new point#get_x;;
- : int = 2
```

`point` クラスは `x` 座標の初期値で抽象化することもできる。

```ocaml
# class point = fun x_init ->
  object
    val mutable x = x_init
    method get_x = x
    method move d = x <- x + d
  end;;
class point :
  int ->
  object val mutable x : int method get_x : int method move : int -> unit end
```

関数定義のように、上記の定義は次のように省略できる。

```ocaml
# class point x_init =
  object
    val mutable x = x_init
    method get_x = x
    method move d = x <- x + d
  end;;
class point :
  int ->
  object val mutable x : int method get_x : int method move : int -> unit end
```

`point` クラスのインスタンスは point オブジェクトを生成するために初期化パラメータをとる関数となる。

```ocaml
# let p = new point;;
val p : int -> point = <fun>

# (p 5)#get_x;;
- : int = 5
```

パラメータ `x_init` はメソッドを含む定義本体全体で参照することができる。
例えば、クラス中のメソッド `get_offset` は初期ポジションに対する相対ポジションを返す。

```ocaml
# class point x_init =
  object
    val mutable x = x_init
    method get_x = x
    method get_offset = x - x_init
    method move d = x <- x + d
  end;;
class point :
  int ->
  object
    val mutable x : int
    method get_offset : int
    method get_x : int
    method move : int -> unit
  end
```

式はクラスのオブジェクト本体の定義より前に評価され束縛される。
これは不変条件を適用するのに便利である。
例えば、ポイントは自動的にグリッド上で最も近いポイントに調整されるとすると、次のようになる。

```ocaml
# class adjusted_point x_init =
  let origin = (x_init / 10) * 10 in
  object
    val mutable x = origin
    method get_x = x
    method get_offset = x - origin
    method move d = x <- x + d
  end;;
class adjusted_point :
  int ->
  object
    val mutable x : int
    method get_offset : int
    method get_x : int
    method move : int -> unit
  end
```

(`x_init` の座標がグリッドじょうでなかった場合に例外を投げることもできる。)
`point`クラスの定義を `origin` の値を指定して呼び出すことで、同じ効果を得ることもできる。

```ocaml
# class adjusted_point x_init = point ((x_init / 10) * 10);;
class adjusted_point : int -> point
```

もう一つの解決策は特別な生成関数に調整処理を定義することである。
```ocaml
# let new_adjusted_point x_init = new point ((x_init / 10) * 10);;
val new_adjusted_point : int -> point = <fun>
```

しかし、先のパターンが一般的により適切であり、それは調整のためのコードがクラス定義の一部になっており、それも継承できるからである。

この機能は他の言語で見られるクラスコンストラクタを提供する。
同じクラスのオブジェクトを生成するためにこの方法でいくつかのコンストラクタを定義することができるが、別の初期化パターンもある。代替手段はイニシャライザを使うことである。
イニシャライザについては3.4章で触れる。

## 3.2 Immediate objects

クラスを介さずに、より直接的にオブジェクトを生成することもできる。

シンタックスはクラスの式と全く同じであるが、結果はクラスではなく単一オブジェクトである。
この章の残りで説明する全ての構造は即時オブジェクトにも適用される。

```ocaml
# let p =
  object
    val mutable x = 0
    method get_x = x
    method move d = x <- x + d
  end;;
val p : < get_x : int; move : int -> unit > = <obj>

# p#get_x;;
- : int = 0

# p#move 3;;
- : unit = ()

# p#get_x;;
- : int = 3
```

式の中に定義することはできないクラスとは違い、即時オブジェクトはどこにでも書けるし、その場にある変数も使える。

```ocaml
# let minmax x y =
  if x < y then object method min = x method max = y end
  else object method min = y method max = x end;;
val minmax : 'a -> 'a -> < max : 'a; min : 'a > = <fun>
```
```ocaml
# let result = minmax 3 9;;
val result : < max : int; min : int > = <obj>
# result#max;;
- : int = 9
# result#min;;
- : int = 3
```

即時オブジェクトはクラスと比較して２つの弱点がある。
１つは即時オブジェクトの型は省略できないこと、もう一つは即時オブジェクトは継承できないことである。
しかし、これら２つの弱点はいくつかのシチュエーションでは強みでもある。それについては、3.3と3.10で述べる。

