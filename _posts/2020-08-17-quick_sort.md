---
layout: post
title:  "クイックソート"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "プログラミング入門 - クイックソートを実装してみよう"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

> クイックソート (quicksort) は、1960年にアントニー・ホーアが開発したソートのアルゴリズム。分割統治法の一種。


[出典: フリー百科事典『ウィキペディア（Wikipedia）』](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BD%E3%83%BC%E3%83%88#:~:text=%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BD%E3%83%BC%E3%83%88%20(quicksort)%20%E3%81%AF%E3%80%81,%E5%88%86%E5%89%B2%E7%B5%B1%E6%B2%BB%E6%B3%95%E3%81%AE%E4%B8%80%E7%A8%AE%E3%80%82)



# アルゴリズム


> 1. 適当な数（ピボット（英語版）という）を選択する（この場合はデータの総数の中央値が望ましい）
> 2. ピボットより小さい数を前方、大きい数を後方に移動させる （分割）
> 3. 二分割された各々のデータを、それぞれソートする

[出典: フリー百科事典『ウィキペディア（Wikipedia）』](https://ja.wikipedia.org/wiki/%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BD%E3%83%BC%E3%83%88#:~:text=%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%BD%E3%83%BC%E3%83%88%20(quicksort)%20%E3%81%AF%E3%80%81,%E5%88%86%E5%89%B2%E7%B5%B1%E6%B2%BB%E6%B3%95%E3%81%AE%E4%B8%80%E7%A8%AE%E3%80%82)

適当な値としては対象となる範囲の中央にある値を選択するようにしました。

ピボットのインデックス = 左のインデックス / 配列の個数 / 2

```rust
let pivot_idx = left + (right - left) / 2;
``` 


1. 要素が１個以下の場合は何もせずに終了
2. ピボットの値を選ぶ
3. 左からピボット以上になる値を探し、見つかった値のインデックスを ```l``` とする。
4. 右からピボット以下になる値を探し、見つかった値のインデックスを ```r``` とする。
5. ```l``` が ```r``` よりも小さい場合は値を交換して ２ に戻る。
   1. ２に戻るときは ```l+1``` から探索開始
   2. ３では ```r-1``` から探索開始する
6. ```r``` が ```l``` 以下になったら範囲を分割する。
   1. 最初から ```l-1``` までと ```r+1``` から最後までの２つの範囲に分割する

<iframe width="560" height="315" src="https://www.youtube.com/embed/_LBUmue8DXI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


# コード

```rust
use env_logger;
use log::{debug, error, info, warn};
use std::env;
extern crate random_vec;
use random_vec::random_vec;

fn quick_sort(list: &mut Vec<u32>, left: usize, right: usize) {
    if left < right {
        let pivot_idx = left + (right - left) / 2;
        let pivot = list[pivot_idx];
        let mut l = left;
        let mut r = right;
        debug!(
            "l={}, r={}, pivot_idx={}, list[pivot]={} ",
            l, r, pivot_idx, pivot
        );

        loop {
            // 左：ピボットより大きい値を探す
            while l <= right && list[l] < pivot {
                // ピボットの値よりも小さい場合はマーカーを右に移動
                l = l + 1;
            }

            // 右：ピボットより小さい値を探す
            while left <= r && pivot < list[r] {
                // ピボットの値よりも大きい場合はマーカを左に移動
                r = r - 1;
            }

            debug!("l={}, list[l]={}", l, list[l]);
            debug!("r={}, list[r]={}", r, list[r]);

            if r <= l {
                break;
            }

            let tmp = list[l];
            list[l] = list[r];
            list[r] = tmp;
            debug!("\x1b[32mswap\x1b[m > list: {:?}", list);

            l = l + 1;
            r = r - 1;
        }
        debug!("\x1b[30m\x1b[47mnext\x1b[m\n");
        if 0 < l {
            quick_sort(list, left, l - 1);
        }
        quick_sort(list, r + 1, right);
    }
}


fn main() {
    // ログ
    env::set_var("RUST_LOG", "info");
    env_logger::init();

    // 配列
    let mut list = [8, 4, 3, 7, 6, 5, 2, 1].to_vec();
    let left = 0;
    let right = list.len() - 1;

    // ソート前
    info!("list: {:?}", list);

    // ソート実行
    quick_sort(&mut list, left, right);

    // ソート後
    info!("sort: {:?}", list);

    info!("\x1b[{}m\x1b[47m== 終了 ==\x1b[m", 30)
}

```
