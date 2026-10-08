# 構文と制御フロー

Rust の構文は C++ と似ていますが、関数型言語の影響を受けており、いくつかの重要な違いがあります。特にさまざまなものが式として評価できる点は、手続き型言語に慣れた人にとっては新しい概念です。この章では、Rust の基本的な構文と制御フローについて説明します。

## ブロック

Rust では C++ と同じように中括弧 `{}` で複数の文を囲むことができます。ですが、C++ と違うのは、ブロックは **式として評価される** ことです。ブロックの最後の式の値がブロック全体の値になります。C++ ではブロックはステートメントであり、値を返しません。

```rust
let x = {
    let y = 10;
    y + 5 // この式の値がブロックの値
};
```

## ループ構文

### While

Rust にも C++ と同じように while ループがあります。条件式を括弧で囲まないのは if と同じです：

```rust
fn main() {
    let mut x = 10;
    while x > 0 {
        println!("Current value: {}", x);
        x -= 1;
    }
}
```

Rust には `do...while` ループはありません。

### Loop

Rust には無限ループを表す `loop` 文があります：

```rust
fn main() {
    loop {
        println!("Just looping");
    }
}
```

### For ループ

Rust にも `for` ループがありますが、少し違います。たとえば配列を一つずつ走査する例を見てみましょう。シンプルな `for` ループは次のようになります：

```rust
fn print_all(v: Vec<i32>) {
    for a in v.iter() {
        println!("{}", a);
    }
}
```

インデックスを使いたい場合（C++の標準的な for ループに近い形）は、次のように書けます：

```rust
fn print_all(v: Vec<i32>) {
    for i in 0..v.len() {
        println!("{}: {}", i, v[i]);
    }
}
```

`..` は範囲演算子です。`0..v.len()` は 0 から `v.len() - 1` までの範囲を生成します。`len` 関数の役割は明らかでしょう。

より Rust らしい書き方は、イテレータを使うことです：

```rust
fn print_all(v: Vec<i32>) {
    for (i, a) in v.iter().enumerate() {
        println!("{}: {}", i, a);
    }
}
```

ここで `enumerate()` はイテレータ `iter()` に接続し、反復中に現在のカウントと要素を生成します。

### Break と Continue

Rust でも C++ と同じように `break` と `continue` が使えます。多重ループで `break` や `continue` を使う場合は、次のようにループにラベルを付けて、どのループから抜けるかを指定できます：

```rust
fn main() {
    'outer: for i in 0..5 {
        for j in 0..5 {
            if i + j > 5 {
                break 'outer; // 外側のループから抜ける
            }
            println!("i: {}, j: {}", i, j);
        }
    }
}
```

## 条件分岐

### If

Rust の `if` 文は C++ と同様に条件分岐を行いますが、いくつかの違いがあります。条件式の括弧は不要で、ブロックの中括弧は必須です。

```rust
if x == 42 {
    println!("x is 42");
}
```

Rust の `if` は式として評価されます。なので C++ の三項演算子 `?:` と同じように使えます。ブロック内の最後の式がセミコロンで終わっていなければ、それがブロックの値になります。ただし True / False のどちらのブロックも同じ型である必要があります。たとえば、次の 2 つの関数は同じ動作をします：

```rust
fn foo(x: i32) -> &'static str {
    let result: &'static str;
    if x < 10 {
        result = "less than 10";
    } else {
        result = "10 or more";
    }
    return result;
}

fn bar(x: i32) -> &'static str {
    if x < 10 {
        "less than 10"
    } else {
        "10 or more"
    }
}
```

最初のコードは C++スタイルの直訳です。2 番目の方が Rust らしい書き方です。

### Match

Rust には C++ の switch 文に似た、でもずっと強力な match 式があります。シンプルなバージョンは見慣れた形でしょう：

```rust
fn print_some(x: i32) {
    match x {
        0 => println!("x is zero"),
        1 => println!("x is one"),
        10 => println!("x is ten"),
        y => println!("x is something else {}", y),
    }
}
```

構文上の違いがいくつかあります。 `条件 => 式` の形（マッチアーム）で条件分岐を記述します。マッチアームは `,` 区切りにします。末尾の `,` は省略可能です。

意味的に重要な違いとして、マッチ式は入力の型を網羅する必要があります。たとえば、上の例では `x: i32` のすべての値をカバーする必要があります。`y => ...` 行を削除してみてください。0、1、10 だけでは `i32` を網羅できないためエラーになります。最後のアームの `y` は、上の条件にマッチしなかった場合に、x が y にバインドされ式が評価されます。

変数に名前を付けたくない場合は、ワイルドカードマッチのように `_` を使えます。何もしない場合は空のブロックを書きます：

```rust
fn print_some(x: i32) {
    match x {
        0 => println!("x is zero"),
        1 => println!("x is one"),
        10 => println!("x is ten"),
        _ => {}
    }
}
```

もう 1 つの意味的な違いは、アーム間のフォールスルーがないことです。`if ... else if ... else` のように動作します。

match はとても強力です。今は値の 'or' 演算子とアームの `if` 句を紹介します。例を見れば分かるでしょう：

```rust
fn print_some_more(x: i32) {
    match x {
        0 | 1 | 10 => println!("x is one of zero, one, or ten"),
        y if y < 20 => println!("x is less than 20, but not zero, one, or ten"),
        y if y == 200 => println!("x is 200 (but this is not very stylish)"),
        _ => {}
    }
}
```

`if` 式と同じく、`match` も式なので、最後の例はこう書き直せます：

```rust
fn print_some_more(x: i32) {
    let msg = match x {
        0 | 1 | 10 => "one of zero, one, or ten",
        y if y < 20 => "less than 20, but not zero, one, or ten",
        y if y == 200 => "200 (but this is not very stylish)",
        _ => "something else",
    };

    println!("x is {}", msg);
}
```

閉じ中括弧の後のセミコロンに注意してください。`let`文はステートメントなので`let msg = ...;`の形式が必要です。右辺は match 式（通常セミコロン不要）ですが、`let` 文にはセミコロンが必要です。これはよく忘れがちです。

> Rust の match 文は Rust の静的解析の肝となる概念です。他の制御構文は、Rust コンパイラの中間表現で loop goto match に変換されます。
