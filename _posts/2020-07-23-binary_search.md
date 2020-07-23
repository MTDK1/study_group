---
layout: post
title:  "二分探索プログラムを作る"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "あなたはプログラマーですか？"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

今回のプログラミング学習は選択ソートプログラムを作ります。

# 二分探索

> ソート済みのリストや配列に入ったデータ（同一の値はないものとする）に対する検索を行うにあたって、 中央の値を見て、検索したい値との大小関係を用いて、検索したい値が中央の値の右にあるか、左にあるかを判断して、片側には存在しないことを確かめながら検索していく。<br>
出典：[二分探索 - Wikipedia](https://ja.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%8E%A2%E7%B4%A2)


# アルゴリズム

1. 与えられた配列を昇順ソートする
2. 配列の中央値と探索する値を比較する
3. 中央値が探索する値と一致すれば、その要素番号を返す
4. 探索する値が中央値よりも小さければ、配列の中央値よりも前の範囲について ２ から実行する
5. 探索する値が中央値よりも大きければ、配列の中央値よりも後ろの範囲について ２ から実行する
6. 探索する値が見つかれば、その要素番号を返す
7. 探索する値が見つからなければ、負の数を返す


# プログラム

```rust
fn binary_search(list: &[i32], value: i32) -> Option<usize> {

    // 探索範囲（開始）
    let mut imin: usize = 0;
    // 探索範囲（終了）
    let mut iend: usize = list.len() - 1;

    // ループ（探索範囲がなくなるまで繰り返す）
    while imin <= iend {

        // 中央の要素番号
        let imid:usize = imin + (iend - imin) / 2;

        // 中央の値と探索値が一致した場合は中央の要素番号を返す
        if list[imid] == value {
            return Some(imid);
        } 
        // 探索値が中央値より小さい場合は探索範囲を左に絞る
        else if value < list[imid] && 0 < imid {
            iend = imid - 1;
        }
        // その他の場合（探索値が中央値より大きい場合）は探索範囲を右に絞る
        else {
            imin = imid + 1;
        }
    }

    // 探索結果が見つからなかった場合は -1 を返す
    return None;

}
fn main() {
    
    // 配列
    let list = [121, 125, 128, 131, 132, 
                137, 138, 141, 145, 151];
    // 探索する値
    // let value = 141;

    for value in 120..154 {
        print!("{} -> ", value);
        // 探索
        match binary_search(&list, value) {
            Some(value) => println!("{}", value),
            None => println!("None")
        }
    }
}
```

```rust
let imid:usize = imin + (iend - imin) / 2;
```

中央値を求める場合、(imin + iend) / 2 と書いても良さそうですが、imin + iend がオーバーフローする可能性があるため上記のようなコードとなっています。


```rust
else if value < list[imid] && 0 < imid {  
  iend = imid - 1;
}
```

ここで 0 < imid という条件があります。 iend は usize 型なので負数を代入するとプログラムはパニックします。
imid が ０ より大きい場合だけ、この条件分岐の中に入るようにします。


# 実行

```bash
$ cargo run
120 -> None
121 -> 0
122 -> None
123 -> None
124 -> None
125 -> 1
126 -> None
127 -> None
128 -> 2
129 -> None
130 -> None
131 -> 3
132 -> 4
133 -> None
134 -> None
135 -> None
136 -> None
137 -> 5
138 -> 6
139 -> None
140 -> None
141 -> 7
142 -> None
143 -> None
144 -> None
145 -> 8
146 -> None
147 -> None
148 -> None
149 -> None
150 -> None
151 -> 9
152 -> None
153 -> None
```

# まとめ

変数宣言で型を明示しました。rust では明示しなくてもコードから推測できるのであれば省略することが可能です。
usize 型を使ったのは、```list.len() - 1``` が usize 型だからです。この値を i32 型に変換すれば ```0 < imid``` この条件が不要になります。

usize 型を i32 型にして実装してみてください。 ヒント ```list.len() - 1``` を型変換するのは try_into を使ってみてください。

```use std::convert::TryInto```<br>
[std::convert::TryInto - Rust](https://doc.rust-lang.org/std/convert/trait.TryInto.html)