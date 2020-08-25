---
layout: post
title:  "シェーカーソート"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "プログラミング入門 - シェーカーソートを実装してみよう"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

> シェーカーソート (英: shaker sort) は、ソートのアルゴリズムの一つ。バブルソートを、効率がよくなるように改良したもの。別名は、双方向バブルソート、改良交換法。
> バブルソートではスキャンを一方向にしか行わないのに対し、シェーカーソートでは交互に二方向に行う。バブルソートと同じく安定な内部ソートで、最悪の場合の時間計算量はO(n2)である。


[出典: フリー百科事典『ウィキペディア（Wikipedia）』](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%82%A7%E3%83%BC%E3%82%AB%E3%83%BC%E3%82%BD%E3%83%BC%E3%83%88)



# アルゴリズム


> バブルソートで1回スキャンを行うと、最後の要素1個がスキャン範囲中最大であることが分かり次回のスキャン範囲を1狭めることができる。さらに、このスキャンの最後で連続してm個の要素の交換が行われていなければ、そのm個についてはソート済みであることが分かるので、次回のスキャン範囲をm狭めることができる。この工夫で、後半が殆ど整列済みのデータに対してバブルソートが高速に行えるようになる。
> シェーカーソートはこれに加え、スキャン方向を毎回反転することにより、スキャン範囲を後方からだけではなく前方からも狭めるようにしたものである。挿入ソートのように、殆ど整列済みのデータに対しては高速に行うことができる。


[出典: フリー百科事典『ウィキペディア（Wikipedia）』](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%82%A7%E3%83%BC%E3%82%AB%E3%83%BC%E3%82%BD%E3%83%BC%E3%83%88)

<iframe width="560" height="315" src="https://www.youtube.com/embed/Oky02gub6_o" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# コード

```rust
pub fn sort(list: &mut Vec<u32>) {
    bubble_sort(list);
}

fn bubble_sort(vec: &mut Vec<u32>) {
    let mut left = 0;
    let mut right = vec.len() - 1;
    let mut last_swap_index = 0;

    loop {
        for i in left..right {
            if vec[i + 1] < vec[i] {
                swap(vec, i, i + 1);
                last_swap_index = i;
            }
        }
        right = last_swap_index;
        if left == right {
            break;
        }

        for i in ((left + 1)..=right).rev() {
            if vec[i - 1] > vec[i] {
                swap(vec, i, i - 1);
                last_swap_index = i;
            }
        }
        left = last_swap_index;
        
        if left == right {
            break;
        }
    }
}

fn swap(vec: &mut Vec<u32>, i: usize, j: usize) {
    let tmp = vec[i];
    vec[i] = vec[j];
    vec[j] = tmp;
}

```