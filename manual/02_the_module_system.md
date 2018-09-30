# The module system

この章では OCaml のモジュールシステムについて紹介する。

## 2.1 Structures

モジュールの主な動機はデータ型の定義とその型に関連する操作を一緒にパッケージングして、これらの定義に一貫した命名規則を適用することである。

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
モジュールをopenすると、モジュールの中身に簡単にアクセスできるようになるが、
どのモジュールに既に定義済みの識別子があるか判別するのが困難になる。
openは現在のスコープ内に存在する識別子をシャドーすることができてしまい、エラーの原因になる。

openはローカルに展開することもできる。
```ocaml
# let open PrioQueue in
  insert empty 1 "hello";;
- : string PrioQueue.queue = Node (1, "hello", Empty, Empty)
```
```ocaml
# PrioQueue.(insert empty 1 "hello");;
- : string PrioQueue.queue = Node (1, "hello", Empty, Empty)
```

2つめの形式で、ローカルな展開のボディが丸括弧、カギカッコ、中括弧で区切られている場合、
ローカルな展開の丸括弧は省略することができる。

```ocaml
# PrioQueue.[empty] = PrioQueue.([empty]);;
- : bool = true

# PrioQueue.[|empty|] = PrioQueue.([|empty|]);;
- : bool = true

# PrioQueue.{ contents = empty } = PrioQueue.({ contents = empty });;
- : bool = true
```
```ocaml
# PrioQueue.[insert empty 1 "hello"];;
- : string PrioQueue.queue list = [Node (1, "hello", Empty, Empty)]
```

`include` ステートメントを使って、他のモジュールの中にモジュールのコンポーネントをコピーすることができる。これは、既存のモジュールを拡張する際に便利である。
```ocaml
# module PrioQueueOpt =
  struct
    include PrioQueue
    let remove_top_opt x =
      try Some(remove_top x) with Queue_is_empty -> None
    let extract_opt x =
      try Some(extract x) with Queue_is_empty -> None
  end;;
module PrioQueueOpt :
  sig
    type priority = int
    type 'a queue =
      'a PrioQueue.queue =
        Empty
      | Node of priority * 'a * 'a queue * 'a queue
    val empty : 'a queue
    val insert : 'a queue -> priority -> 'a -> 'a queue
    exception Queue_is_empty
    val remove_top : 'a queue -> 'a queue
    val extract : 'a queue -> priority * 'a * 'a queue
    val remove_top_opt : 'a queue -> 'a queue option
    val extract_opt : 'a queue -> (priority * 'a * 'a queue) option
  end
```

## 2.2 Signatures

シグネチャは構造体のインタフェースである。シグネチャは構造体のどのコンポーネントが外からアクセス可能か指定する。これを使ってローカル関数などのコンポーネントを隠蔽することができる。もしくは、コンポーネントに制限された型を付けて公開できる。

次の例は、補助関数である `remove_top` をシグネチャに記述せず隠蔽している。
```ocaml
# module type PRIOQUEUE =
  sig
    type priority = int (* still concrete *)
    type 'a queue       (* now abstract *)
    val empty : 'a queue
    val insert : 'a queue -> int -> 'a -> 'a queue
    val extract : 'a queue -> int * 'a * 'a queue
    exception Queue_is_empty
  end;;
module type PRIOQUEUE =
  sig
    type priority = int
    type 'a queue
    val empty : 'a queue
    val insert : 'a queue -> int -> 'a -> 'a queue
    val extract : 'a queue -> int * 'a * 'a queue
    exception Queue_is_empty
  end
```
```ocaml
# module AbstractPrioQueue = (PrioQueue: PRIOQUEUE);;
module AbstractPrioQueue : PRIOQUEUE

# AbstractPrioQueue.remove_top;;
Error: Unbound value AbstractPrioQueue.remove_top

# AbstractPrioQueue.insert AbstractPrioQueue.empty 1 "hello";;
- : string AbstractPrioQueue.queue = <abstr>
```

この制限は構造体の定義時にも指定できる。
```ocaml
module PrioQueue = (struct ... end : PRIOQUEUE)
```
```ocaml
module PrioQueue : PRIOQUEUE = struct ... end;;
```

シグネチャも構造体と同様にコピーすることができる。
```ocaml
# module type PRIOQUEUE_WITH_OPT =
  sig
    include PRIOQUEUE
    val extract_opt : 'a queue -> (int * 'a * 'a queue) option
  end;;
module type PRIOQUEUE_WITH_OPT =
  sig
    type priority = int
    type 'a queue
    val empty : 'a queue
    val insert : 'a queue -> int -> 'a -> 'a queue
    val extract : 'a queue -> int * 'a * 'a queue
    exception Queue_is_empty
    val extract_opt : 'a queue -> (int * 'a * 'a queue) option
  end
```

## 2.3 Functors

ファンクタはモジュールからモジュールへの関数である。
ファンクターをつかうことで、パラメータ化されたモジュールを作ることができ、特定の実装を得るためのパラメータとして他のモジュールを与えることができる。
例えば、 set を sorted list として実装されている `Set` モジュールは、要素
の型と比較関数である `compare` を提供するモジュールで動作するようにパラメータ化できる。

```ocaml
# type comparison = Less | Equal | Greater;;
type comparison = Less | Equal | Greater

# module type ORDERED_TYPE =
  sig
    type t
    val compare: t -> t -> comparison
  end;;
module type ORDERED_TYPE = sig type t val compare : t -> t -> comparison end
```
```ocaml
# module Set =
functor (Elt: ORDERED_TYPE) ->
  struct
    type element = Elt.t
    type set = element list
    let empty = []
    let rec add x s =
      match s with
        [] -> [x]
      | hd :: tl ->
        match Elt.compare x hd with
          Equal -> s
        | Less  -> x :: s
        | Greater -> hd :: add x tl
    let rec member x s =
      match s with
        [] -> false
      | hd :: tl ->
        match Elt.compare x hd with
          Equal -> true
        | Less  -> false
        | Greater -> member x tl
  end;;
module Set :
  functor (Elt : ORDERED_TYPE) ->
    sig
      type element = Elt.t
      type set = element list
      val empty : 'a list
      val add : Elt.t -> Elt.t list -> Elt.t list
      val member : Elt.t -> Elt.t list -> bool
    end
```

ordered 型を実装した構造体に `Set` ファンクタを適用することによって、
この型の set 演算をえることができる。

```ocmal
# module OrderedString =
  struct
    type t = string
    let compare x y = if x = y then Equal else if x < y then Less else Greater
  end;;
module OrderedString :
  sig type t = string val compare : 'a -> 'a -> comparison end
```
```
# module StringSet = Set(OrderedString);;
module StringSet :
  sig
    type element = OrderedString.t
    type set = element list
    val empty : 'a list
    val add : OrderedString.t -> OrderedString.t list -> OrderedString.t list
    val member : OrderedString.t -> OrderedString.t list -> bool
  end

# StringSet.member "bar" (StringSet.add "foo" StringSet.empty);;
- : bool = false
```

## 2.4 Functors and type abstraction

`PrioQueue`の例のように、`set`型の実際の実装を隠すのは良いスタイルであり、構造体のユーザーはセットがリストであることに依存せずに済み、あとでより効率的なセットの実装に変更する際もユーザーのコードを壊さずに済む。
適切なファンクタのシグネチャによって制限することによって実現できる。

```ocaml
# module type SETFUNCTOR =
  functor (Elt: ORDERED_TYPE) ->
    sig
      type element = Elt.t
      type set
      val empty: set
      val add: element -> set -> set
      val member: element -> set -> bool
    end;;
module type SETFUNCTOR =
  functor (Elt : ORDERED_TYPE) ->
    sig
      type element = Elt.t
      type set
      val empty : set
      val add : element -> set -> set
      val member : element -> set -> bool
    end
```
```ocaml
# module AbstractSet = (Set: SETFUNCTOR);;
module AbstractSet : SETFUNCTOR

# module AbstractStringSet = AbstractSet(OrderedString);;
module AbstractStringSet :
  sig
    type element = OrderedString.t
    type set = AbstractSet(OrderedString).set
    val empty : set
    val add : element -> set -> set
    val member : element -> set -> bool
  end

# AbstractStringSet.add "gee" AbstractStringSet.empty;;
- : AbstractStringSet.set = <abstr>
```

よりエレガントに上記の型制約を書こうとすると、ファンクタによって返された構造体のシグネチャに名前をつけ、そのシグネチャを制約に使用することができる。

```ocaml
# module type SET =
  sig
    type element
    type set
    val empty: set
    val add: element -> set -> set
    val member: element -> set -> bool
  end;;
module type SET =
  sig
    type element
    type set
    val empty : set
    val add : element -> set -> set
    val member : element -> set -> bool
  end
```
```ocaml
# module WrongSet = (Set: functor(Elt: ORDERED_TYPE) -> SET);;
module WrongSet : functor (Elt : ORDERED_TYPE) -> SET

# module WrongStringSet = WrongSet(OrderedString);;
module WrongStringSet :
  sig
    type element = WrongSet(OrderedString).element
    type set = WrongSet(OrderedString).set
    val empty : set
    val add : element -> set -> set
    val member : element -> set -> bool
  end

# WrongStringSet.add "gee" WrongStringSet.empty;;
Error: This expression has type string but an expression was expected of type
         WrongStringSet.element = WrongSet(OrderedString).element
```

`SET` は `element` の型を抽象的に指定しており、そのためファンクタの結果による `element` と引数の `t` が忘れられることが問題である。その結果、 `WrongStringSet.element` は `string` と方が異なり、 `WrongStringSet` の演算は `string` に適用することができない。
`element` の型がシグネチャ `SET` の中で `Elt.t` として宣言されていることが大切であり、 上の例では `Elt` が存在しない状態で定義されており `Elt.t` として宣言することができない。
これを解決するために、OCaml は追加の型同一性をシグネチャにエンリッチするための `with　type`  を提供している。

```ocaml
# module AbstractSet2 =
  (Set: functor(Elt: ORDERED_TYPE) -> (SET with type element = Elt.t));;
module AbstractSet2 :
  functor (Elt : ORDERED_TYPE) ->
    sig
      type element = Elt.t
      type set
      val empty : set
      val add : element -> set -> set
      val member : element -> set -> bool
    end
```

この簡単な構造体のケースのようにファンクタを定義しその結果を制限するために別のシンタックスが提供されている。
```ocaml
module AbstractSet2(Elt: ORDERED_TYPE): (SET with type element = Elt.t) =
  struct ... end;;
```

ファンクタの結果に対して型コンポーネントを抽象化することは、高度な型安全性を提供する強力なテクニックである。

たとえば、一般的な `OrderedString` と異なる実装を持つ文字列の順序付けを考える。
次の例では、文字列の大文字小文字を区別せずに比較する文字列を定義する。
```ocaml
# module NoCaseString =
  struct
    type t = string
    let compare s1 s2 =
      OrderedString.compare (String.lowercase_ascii s1) (String.lowercase_ascii s2)
  end;;
module NoCaseString :
  sig type t = string val compare : string -> string -> comparison end
```
```ocaml
# module NoCaseStringSet = AbstractSet(NoCaseString);;
module NoCaseStringSet :
  sig
    type element = NoCaseString.t
    type set = AbstractSet(NoCaseString).set
    val empty : set
    val add : element -> set -> set
    val member : element -> set -> bool
  end
```
```ocaml
# NoCaseStringSet.add "FOO" AbstractStringSet.empty;;
Error: This expression has type
         AbstractStringSet.set = AbstractSet(OrderedString).set
       but an expression was expected of type
         NoCaseStringSet.set = AbstractSet(NoCaseString).set
```

AbstractStringSet.set　と NoCaseStringSet.set は互換性がなく、これら２つの型の値は一致しない。たとえ、どちらのセット型も同じ型の要素を含んでいても、それらは異なる Orderingの型でビルドされており、異なる不変条件は操作によって維持されなければならない。
 `AbstractStringSet` の操作を `NoCaseStringSet.set` の値に適用することは誤った結果を返すことになるか、`NoCaseStringSet` の不変条件を破ったリストを作ることになる。


## 2.5 Modules and separate compilation

これまでの全てのモジュールの例は、REPL上における例であったが、モジュールは大きくて、まとめてコンパイルするようなプログラムに有効である。ソースをコンパイル単位にいくつかのファイルに分割し、個別にコンパイルすることで変更後の再コンパイルを必要最低限にできる。

OCaml ではコンパイル単位は構造体とシグネチャの当別なケースであり、ユニット間の関係はモジュールシステムの観点から容易に説明できる。
コンパイルユニットAは2つのファイルから構成される。
- 実装ファイル `A.ml` は一連の定義を含み、 `struct...end` を構成することに似ている。
- インタフェースファイル `A.mli` は一連の仕様を含み、 `sig...end` を構成することに似ている。

次のような定義がトップレベルで入力された場合と同様に、これらの２つのファイルはともに `A` と名付けられた構造を定義する。
```ocaml
module A: sig (* contents of file A.mli *) end
  = struct (* contents of file A.ml *) end;;
```

コンパイルユニットを定義するファイルは `ocamlc -c` コマンドを使うことで、個別にコンパイルできる。(`-c` オプションはコンパイルのみ行い、リンクは行わない意味である。)
これによって、インタフェースファイル(*.cmi)とオブジェクトコードファイル(*.cmo)が生成される。全てのユニットがコンパイルされた時、それらの .cmoファイルは `ocamlc` コマンドをを使うことで、一緒にリンクされる。

例えば、次のコマンドは２つのコンパイルユニット `Aux` と `Main` から構成されるプログラムをコンパイル & リンクする。
```shell
$ ocamlc -c Aux.mli
$ ocamlc -c Aux.ml
$ ocamlc -c Main.mli
$ ocamlc -c Main.ml
$ ocamlc -o theprogram Aux.cmo Main.cmo
```

`Main` は `Aux` を参照することができ、 `Main.ml` と `Main.mli` に含まれる定義と宣言は、`Aux.ml` 内の定義を `Aux.ident` 記法を使うことで参照することができ、これらの定義は `Aux.mli` でエクスポートされる。

トップレベルの構造体だけ個別にコンパイルされたファイルにマップでき、ファンクタもモジュールタイプもマップできないことに注意する。
しかし、全てのモジュールクラスオブジェクトは構造体のコンポーネントとして現れることがあるので、そのためにファンクタまたはモジュールタイプを構造体の中に配置することでファイルにマップできる。
