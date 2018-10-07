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

## 3.3 Reference to self

メソッドまたはイニシャライザは自分のメソッドを実行することができる。
そのため、自身は明示的に束縛されなければならない。次の例では `s` を使っているが、識別子は何でもよく、 `self` がよく使われる。
```ocaml
# class printable_point x_init =
  object(s)
    val mutable x = x_init
    method get_x = x
    method move d = x <- x + d
    method print = print_int s#get_x
  end;;
class printable_point :
  int ->
  object
    val mutable x : int
    method get_x : int
    method move : int -> unit
    method print : unit
  end
```
```ocaml
# let p = new printable_point 7;;
val p : printable_point = <obj>

# p#print;;
7- : unit = ()
```

動的にメソッドの実行時に変数 `s` は束縛される。
実際にクラス `printable_point` が継承された時、変数 `s` はサブクラスのオブジェクトに正しく束縛される。

self の一般的な問題は、selfの型はサブクラスの中で拡張されるが、事前にそれを修正することはできない。
簡単な例は次の通り。
```ocaml
# let ints = ref [];;
val ints : '_weak1 list ref = {contents = []}
```
```ocaml
# class my_int =
  object (self)
    method n = 1
    method register = ints := self :: !ints
  end;;
Error: This expression has type < n : int; register : 'a; .. >
       but an expression was expected of type 'weak1
       Self type cannot escape its class
```

self を外部参照に入れることは、継承によって拡張することを不可能にする。この問題の回避方法は3.12でお目にかかれる。即時オブジェクトは拡張できないため、このような問題は起きない。
```ocaml
# let my_int =
  object (self)
    method n = 1
    method register = ints := self :: !ints
  end;;
val my_int : < n : int; register : unit > = <obj>
```

## 3.4 Initializers

クラス定義の中の let 束縛はオブジェクトが生成される前に評価される。
イニシャライザをつかうことで、オブジェクトが構築された直後に式を評価することもできる。イニシャライザは self とインスタンス変数にアクセスできる。

```ocaml
# class printable_point x_init =
  let origin = (x_init / 10) * 10 in
  object (self)
    val mutable x = origin
    method get_x = x
    method move d = x <- x + d
    method print = print_int self#get_x
    initializer print_string "new point at "; self#print; print_newline ()
  end;;
class printable_point :
  int ->
  object
    val mutable x : int
    method get_x : int
    method move : int -> unit
    method print : unit
  end
```
```ocaml
# let p = new printable_point 17;;
new point at 10
val p : printable_point = <obj>
```

イニシャライザはオーバーライドできない。全てのイニシャライザが一つずつ順に評価される。イニシャライザは不変条件を強制するのに特に便利である。

## 3.5 Virtual methods

`virtual` キーワードを使うことで、実定義をせずにメソッドを宣言することができる。
このメソッドはサブクラスによって実装を与えられる。
仮想メソッドを含むクラスは `virtual` フラグをつける必要があり、それはインスタンス化できない。

```ocaml
# class virtual abstract_point x_init =
  object (self)
    method virtual get_x : int
    method get_offset = self#get_x - x_init
    method virtual move : int -> unit
  end;;
class virtual abstract_point :
  int ->
  object
    method get_offset : int
    method virtual get_x : int
    method virtual move : int -> unit
  end
```
```ocaml
# new abstract_point 1;;
Error: Cannot instantiate the virtual class abstract_point
```
```ocaml
# class point x_init =
  object
    inherit abstract_point x_init
    val mutable x = x_init
    method get_x = x
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
```ocaml
# new point 1;;
- : point = <obj>
```

メソッドと同じようにインスタンス変数も `virtual` として宣言でき、

```ocaml
# class virtual abstract_point2 =
  object
    val mutable virtual x: int
    method move d = x <- x + d
  end;;
class virtual abstract_point2 :
  object val mutable virtual x : int method move : int -> unit end
```
```ocaml
# class point2 x_init =
  object
    inherit abstract_point2
    val mutable x = x_init
    method get_oggset = x - x_init
  end;;
class point2 :
  int ->
  object
    val mutable x : int
    method get_oggset : int
    method move : int -> unit
  end
```

## 3.6 Private methods

プライベートメソッドはオブジェクトのインタフェースに現れないメソッドである。プライベートメソッドは同オブジェクトの他のメソッドからしか実行できない。

```ocaml
# class restricted_point x_init =
  object (self)
    val mutable x = x_init
    method get_x = x
    method private move d = x <- x + d
    method bump = self#move 1
  end;;
class restricted_point :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method private move : int -> unit
  end
```
```ocaml
# let p = new restricted_point 0;;
val p : restricted_point = <obj>

# p#move 10;;
Error: This expression has type restricted_point
       It has no method move

# p#bump;;
- : unit = ()

# p#get_x;;
- : int = 1
```

同じクラスの他のオブジェクトから呼ぶことのできるJavaやC++のプライベートメソッドやプロテクテッドメソッドとは同じものではないことに注意する。

これはOCamlが型とクラスを独立させていることの直接的な影響である。
２つの無関係のクラスが同じ型のオブジェクトを生成するため、特定のクラスからオブジェクトが生成されることを型レベルで保証することはできない。

プライベートメソッドはシグネチャのマッチングで隠蔽されていない限りは継承される。
(デフォルトではサブクラスでは visible になる)

プライベートメソッドはサブクラスでは公開することができる。
```ocaml
# class point_again x =
    object (self)
      inherit restricted_point x
      method virtual move : _
    end;;
class point_again :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method move : int -> unit
  end
```

`virtual` アノテーションは、その定義を提供せずメソッドに言及する場合にのみ使用される。`private` アノテーションを追加しなかったことで、もとの定義を維持したままメソッドはパブリックになった。
他の表現は次のようになる。
```ocaml
# class point_again x =
  object (self: < move : _; ..>)
    inherit restricted_point x
  end;;
class point_again :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method move : int -> unit
  end
```
selfの型の制約はパブリックな `move` メソッドを要求しており、 `private` をオーバーライドするならこれで十分である。

サブクラスでもプライベートメソッドはプライベートのままであるべきと考えることもできるが、プライベートメソッドはサブクラスで visible であるため、そのメソッドを実行する同じ名前のメソッドを定義することができる。
もう一つの重い解決方法は次のようになる。

```ocaml
# class point_again x =
  object
    inherit restricted_point x as super
    method move = super#move
  end;;
class point_again :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method move : int -> unit
  end
```

もちろんプライベートメソッドも `virtual` にできる。そのとき、キーワードは `method private virtual` の順で記述する必要がある。

## 3.7 Class interface

クラスインタフェースはクラス定義から推論される。
クラスインタフェースは直接定義され、クラスの型を限定するために使われる。
クラス宣言のようにクラスインタフェースも新しい省略型を定義する。

```ocaml
# class type restricted_point_type =
  object
    method get_x: int
    method bump : unit
  end;;
class type restricted_point_type =
  object method bump : unit method get_x : int end
```
```ocaml
# function (x: restricted_point_type) -> x;;
- : restricted_point_type -> restricted_point_type = <fun>
```

クラスインタフェースを使ってはクラスの型を制約することができる。具象インスタンス変数と具象プライベートメソッドはクラス型制約によって隠すことができるが、パブリックメソッドや仮想メンバーはできない。
```ocaml
# class restricted_point' x = (restricted_point x: restricted_point_type);;
class restricted_point' : int -> restricted_point_type
```
または、
```ocaml
# class restricted_point' = (restricted_point : int -> restricted_point_type);;
class restricted_point' : int -> restricted_point_type
```

クラスのインタフェースはモジュールのシグネチャで指定することもでき、
推論されるモジュールのシグネチャを制限するために使用できる。
```ocaml
# module type POINT = sig
  class restricted_point' : int ->
    object
      method get_x : int
      method bump : unit
    end
  end;;
module type POINT =
  sig
    class restricted_point' :
      int -> object method bump : unit method get_x : int end
  end
```
```ocaml
# module Point: POINT = struct
    class restricted_point' = restricted_point
  end;;
module Point : POINT
```

## 3.8 Inheritance

`point` を継承した `colored_point` を定義することで継承について示す。
このクラスは `point` クラスの全てのインスタンス変数とメソッドを持っており、加えて新しいインスタンス変数 `c` と新しいメソッド `color` を持っている。
```ocaml
# class colored_point x (c:string) =
  object
    inherit point x
    val c = c
    method color = c
  end;;
class colored_point :
  int ->
  string ->
  object
    val c : string
    val mutable x : int
    method color : string
    method get_offset : int
    method get_x : int
    method move : int -> unit
  end
```
```ocaml
# let p' = new colored_point 5 "red";;
val p' : colored_point = <obj>

# p'#get_x, p'#color;;
- : int * string = (5, "red")
```
`point` は `color` メソッドを持っていないため、 `colored_point` とは互換性が無い。しかし、次に示す`get_succ_x` 関数は、`get_x` メソッドを持つ任意のオブジェクト `p` にメソッド `get_x` を適用するジェネリック関数である。従って、 `get_succ_x` は `point` と `colored_point` 両方に適用される。
```ocaml
# let get_succ_x p = p#get_x + 1;;
val get_succ_x : < get_x : int; .. > -> int = <fun>

# get_succ_x p + get_succ_x p';;
- : int = 9
```

メソッドは事前に宣言される必要はない。次の例は未定義の関数 `set_x` を持つ任意のオブジェクトを引数としてとる。
```ocaml
# let set_x p = p#set_x;;
val set_x : < set_x : 'a; .. > -> 'a = <fun>

# let incr p = set_x p (get_succ_x p);;
val incr : < get_x : int; set_x : int -> 'a; .. > -> 'a = <fun>
```

## 3.9 Multiple inheritance

多重継承は許されている。最後に定義されたメソッドのみが維持される。
親クラスで visible なメソッドのサブクラスにおける再定義は親クラスの定義をオーバーライドする。メソッドの以前の定義は関連する祖先としてバインドすることで再利用することができる。 以下では、`super` は祖先である `printable_point` にバインドされている。`super` という名前はスーパークラスのメソッドを呼ぶためのみに使われる疑似値識別子である。
```ocaml
# class printable_colored_point y c =
  object (self)
    val c = c
    method color = c
    inherit printable_point y as super
    method! print =
      print_string "(";            ^R
      print_string "(";
      super#print;
      print_string ",";
      print_string (self#color);
      print_string ")"
  end;;
class printable_colored_point :
  int ->
  string ->
  object
    val c : string
    val mutable x : int
    method color : string
    method get_x : int
    method print : unit
  end
```
```ocaml
# let p' = new printable_colored_point 17 "red";;
new point at (10,red)
val p' : printable_colored_point = <obj>

# p'#print;;
(10,red)- : unit = ()
```

親クラスの隠されたプライベートメソッドはもはや visible ではなくなり、オーバーライドされない。イニシャライザはプライベートメソッドとして扱われ、クラス階層に沿った全てのイニシャライザが継承順に評価される。

わかりやすくするために、メソッド `print` は `method` キーワードに `!` を付けることで、別の定義を上書きすることを明示している。もし、メソッド `print` が `printable_point` の `print` メソッドをオーバーライドしていなかったら、コンパイラはエラーをあげる。
```ocaml
# object
    method! m = ()
  end;;
Error: The method `m' has no previous definition
```

この明示的なオーバーライドアノテーションは `val` と `inherit` でも使える。
```ocaml
# class another_printable_colored_point y c c' =
    object (self)
    inherit printable_point y
    inherit! printable_colored_point y c
    val! c = c'
  end;;
class another_printable_colored_point :
  int ->
  string ->
  string ->
  object
    val c : string
    val mutable x : int
    method color : string
    method get_x : int
    method print : unit
  end
```
