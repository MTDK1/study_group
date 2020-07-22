---
layout: post
title:  "線形探索プログラムを作る"
author: mtdk1
categories: [ プログラミング入門 ]
tags: [Rust]
description: "あなたはプログラマーですか？"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

今回のプログラミング学習は線形探索をするプログラムを作ります。

# 線形探索

> 線形探索（せんけいたんさく、英: linear search, sequential search）は、検索のアルゴリズムの一つ。 リストや配列に入ったデータに対する検索を行うにあたって、 先頭から順に比較を行い、それが見つかれば終了する。
出典:[線型探索 - Wikipedia](https://ja.wikipedia.org/wiki/%E7%B7%9A%E5%9E%8B%E6%8E%A2%E7%B4%A2)

1. 探索対象となる配列を用意します
2. 探索する値を用意します
3. 線形探索の関数の配列と探索する値を渡します
4. 線形探索の関数は Option 型の値を返します
   1. 探索する値が配列の中に見つかった場合は、配列の要素番号を返します
   2. 見つからなかった場合は None を返します
5. 返却された値をパターンマッチによって処理を分けます
   1. 値が返って来たならば、その値を画面に出力
   2. None の場合は画面に None を出力


 # プログラム

```rust
fn linear_search(list: &[i32], value: i32) -> Option<usize> {

    for (index, item) in (0..).zip(list.iter()) {
        if *item == value {
            return Some(index);
        }
    }
    None
}


fn main() {
    
    let list = [6, 3, 7, 2, 8, 1, 4, 5, 9];
    let value = 4;

    match linear_search(&list, value) {
        Some(idx) => println!("value: {}, index = {}", value, idx),
        None => println!("None")
    }
}

```

全て main 関数内に収めることができますが、今回は線形探索を行う関数を作成しました。
繰り返し処理では要素番号を知るために zip 関数を使用しました。
返却する値は値が見つかれば、その要素番号、見つからなければ負の値とするのが多くのサイトで紹介されていますが、
今回は Option 型を返却するようにしました。値が見つかれば Some(値)、見つからなければ None が返却されます。
コードがみやすくなった気がします。返却された値によって分岐するのにパターンマッチを使用しました。これも rust らしい書き方ではないでしょうか

# 実行

```bash
$ cargo run
value: 4, index = 6
```

# まとめ

探索する対象が i32 型の配列でした。数を対象として探索するのは非常に簡単に実装できると思います。
数ではなく文字列だったらどうでしょう？ぜひ実装してみてください。
