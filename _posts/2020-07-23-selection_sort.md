---
layout: post
title:  "選択ソートプログラムを作る"
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

# 選択ソート

> 選択ソート（英: selection sort）は、ソートのアルゴリズムの一つ。配列された要素から、最大値やまたは最小値を探索し配列最後の要素と入れ替えをおこなうこと。最悪計算時間がO(n2)と遅いが、アルゴリズムが単純で実装が容易なため、しばしば用いられる。<br>
出典:[選択ソート - Wikipedia](https://ja.wikipedia.org/wiki/%E9%81%B8%E6%8A%9E%E3%82%BD%E3%83%BC%E3%83%88#:~:text=%E9%81%B8%E6%8A%9E%E3%82%BD%E3%83%BC%E3%83%88%EF%BC%88%E8%8B%B1%3A%20selection%20sort,%E3%81%AA%E3%81%9F%E3%82%81%E3%80%81%E3%81%97%E3%81%B0%E3%81%97%E3%81%B0%E7%94%A8%E3%81%84%E3%82%89%E3%82%8C%E3%82%8B%E3%80%82)

# アルゴリズム

> データ列中で一番小さい値を探し、1番目の要素と交換する。次に、2番目以降のデータ列から一番小さい値を探し、2番目の要素と交換する。これを、データ列の最後まで繰り返す（厳密には、データ列の最後より1つ手前までの繰り返しでよい。一つ前まで交換済みであれば、最後（残り）は必ず最大値になるからである）。大小が入れ替わる降順の場合も同様の手法である。<br>
> 出典: [選択ソート - Wikipedia](https://ja.wikipedia.org/wiki/%E9%81%B8%E6%8A%9E%E3%82%BD%E3%83%BC%E3%83%88])

例

- 4, 3, 1, 9, 2, 6, 5, 8, 7
  - データ列の１番目の要素： 4
  - データ列の中で一番小さい値: 1（３番目の要素）
  - １番目の要素と３番目の要素を交換する
    - 交換前：4, 3, 1, 9, 2, 6, 5, 8, 7
    - 交換後：1, 3, 4, 9, 2, 6, 5, 8, 7
- 1, 3, 4, 9, 2, 6, 5, 8, 7
  - データ列の２番目以降で一番小さい値：2（５番目の要素）
  - ２番目の要素と５番目の要素を交換する
    - 交換前：1, 3, 4, 9, 2, 6, 5, 8, 7
    - 交換後：1, 2, 4, 9, 3, 6, 5, 8, 7
- 1, 2, 4, 9, 3, 6, 5, 8, 7
  - データ列の３番目以降で一番小さい値：3（５番目の要素）
  - ３番目の要素と５番目の要素を交換する
    - 交換前：1, 2, 4, 9, 3, 6, 5, 8, 7
    - 交換後：1, 2, 3, 9, 4, 6, 5, 8, 7
- 1, 2, 3, 9, 4, 6, 5, 8, 7
  - データ列の４番目以降で一番小さい値：4（５番目の要素）
  - ４番目の要素と５番目の要素を交換する
    - 交換前：1, 2, 3, 9, 4, 6, 5, 8, 7
    - 交換後：1, 2, 3, 4, 9, 6, 5, 8, 7
- 1, 2, 3, 4, 9, 6, 5, 8, 7
  - データ列の５番目以降で一番小さい値：5（７番目の要素）
  - ５番目の要素と７番目の要素を交換する
    - 交換前：1, 2, 3, 4, 9, 6, 5, 8, 7
    - 交換後：1, 2, 3, 4, 5, 6, 9, 8, 7


最後から１つ前の８番目の要素まで繰り返す

- 1, 2, 3, 4, 5, 6, 7, 8, 9
  - データ列の８番目以降で一番小さい値：８（８番目の要素）
  - 交換する必要なし

ソート完了

1, 2, 3, 4, 5, 6, 7, 8, 9



# プログラム

```rust
fn main() {

    let mut list = vec![4, 3, 1, 9, 2, 6, 5, 8, 7];
    println!("{:?}", list);

    for i in 0..(list.len() - 1) {
        let mut min = list[i];
        let mut index = i;
        for j in (i + 1)..list.len() {
            if list[j] < min {
                min = list[j];
                index = j;
            }
        }
        if i != index  {
            list[index] = list[i];
            list[i] = min;
            println!("> {:?}", list);
        }
    }

    println!("{:?}", list);   
}
```


# 実行

```bash
$ cargo run
[4, 3, 1, 9, 2, 6, 5, 8, 7]
> [1, 3, 4, 9, 2, 6, 5, 8, 7]
> [1, 2, 4, 9, 3, 6, 5, 8, 7]
> [1, 2, 3, 9, 4, 6, 5, 8, 7]
> [1, 2, 3, 4, 9, 6, 5, 8, 7]
> [1, 2, 3, 4, 5, 6, 9, 8, 7]
> [1, 2, 3, 4, 5, 6, 7, 8, 9]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

# まとめ

Wikipedia で紹介されている実装例の１つ目は最小値を探すという部分がありませんがソートはできています。
比較する回数は同じですが、値を交換する回数が多くなっていますね。
値の交換は簡単に見えますが、計算時間が無視できなくなる場合があります。
アルゴリズムを実装するときは計算時間が短くなるようにしましょう。