---
layout: post
title:  "バブルソート"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "プログラミング入門 - バブルソートを実装してみよう"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

与えられた配列を昇順、降順で並べ替えます。

# コード

```rust
fn bubble_sort(vec: &mut Vec<u32>) {
    for i in (1..vec.len()).rev() {     // -----   Loop 1
        for j in 1..=i {                // Loop 2 
            if vec[j] < vec[j - 1] {    
                let tmp = vec[j - 1];   
                vec[j - 1] = vec[j];    
                vec[j] = tmp;           
            }                           
        }                               // Loop 2   
    }                                   // -----   Loop 1
}
```

# 例

```5, 0, 2, 1, 3, 4``` ６個の値をバブルソートに与える。

{:refdef: style="text-align: center;"}
![Fig.2]({{site.baseurl}}/assets/images/bubblesort/bs002.svg)
{: refdef}

Loop 1 の ```(2..vec.len()).rev()``` は [5, 4, 3, 2, 1] です。

## １回目

{:refdef: style="text-align: center;"}
![Fig.3]({{site.baseurl}}/assets/images/bubblesort/bs003.svg)
{: refdef}

赤枠は Loop 2 の j が変化するごとに右に移動します。赤枠にある条件式が成立する場合は中身を入れ替えます。


{:refdef: style="text-align: center;"}
![Fig.1]({{site.baseurl}}/assets/images/bubblesort/bs001.svg)
{: refdef}

一番右、i = 5 の値が最大値となりました。

## ２回目

{:refdef: style="text-align: center;"}
![Fig.4]({{site.baseurl}}/assets/images/bubblesort/bs004.svg)
{: refdef}

Loop 2 を抜けて、i = 4 になりました。<br>
赤枠 j は 1, 2, 3, 4 と変化します。


{:refdef: style="text-align: center;"}
![Fig.5]({{site.baseurl}}/assets/images/bubblesort/bs005.svg)
{: refdef}

## 3回目

{:refdef: style="text-align: center;"}
![Fig.6]({{site.baseurl}}/assets/images/bubblesort/bs006.svg)
{: refdef}

Loop 2 を抜けて、i = 3 になりました。<br>
赤枠 j は 1, 2, 3 と変化します。

## 4回目

{:refdef: style="text-align: center;"}
![Fig.7]({{site.baseurl}}/assets/images/bubblesort/bs007.svg)
{: refdef}

Loop 2 を抜けて、i = 2 になりました。<br>
赤枠 j は 1, 2 と変化します。

## 5回目

{:refdef: style="text-align: center;"}
![Fig.8]({{site.baseurl}}/assets/images/bubblesort/bs008.svg)
{: refdef}

Loop 2 を抜けて、i = 1 になりました。<br>
赤枠 j は 1 だけになり、Loop 2, Loop 1 を抜けます。

## 結果

{:refdef: style="text-align: center;"}
![Fig.9]({{site.baseurl}}/assets/images/bubblesort/bs009.svg)
{: refdef}

# 実行

```rust
fn main() {
    let mut vec: Vec<u32> = [16, 18, 6, 17, 0, 11, 9, 12, 10, 8, 2, 14, 13, 3, 15, 4, 19, 5, 1, 7].to_vec();

    println!("ソート前: {:?}", vec);
    bubble_sort(&mut vec);
    println!("ソート後: {:?}", vec);
}
```

```bash
$ cargo run

ソート前: [16, 18, 6, 17, 0, 11, 9, 12, 10, 8, 2, 14, 13, 3, 15, 4, 19, 5, 1, 7]
ソート後: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

# おまけ

Cargo.toml に下記を追加
```toml
[dependencies]
rand = "0.7.3"
```

ランダムになった配列を作成する

```rust
use rand::seq::SliceRandom;
use rand::thread_rng;

fn random_vec(length: u32) -> Vec<u32> {
    let mut vec: Vec<u32> = (0..length).collect();
    vec.shuffle(&mut thread_rng());
    vec
}
```

