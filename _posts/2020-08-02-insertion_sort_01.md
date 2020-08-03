---
layout: post
title:  "挿入ソート"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "プログラミング入門 - 挿入を実装してみよう"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

与えられた配列を昇順、降順で並べ替えます。

{:refdef: style="text-align: center;"}
![Fig.1]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort001.svg)
{: refdef}


# アルゴリズム

> 挿入ソート（インサーションソート）は、ソートのアルゴリズムの一つ。整列してある配列に追加要素を適切な場所に挿入すること。<br>
> [挿入ソート - Wikipedia](https://ja.wikipedia.org/wiki/%E6%8C%BF%E5%85%A5%E3%82%BD%E3%83%BC%E3%83%88#:~:text=%E6%8C%BF%E5%85%A5%E3%82%BD%E3%83%BC%E3%83%88%EF%BC%88%E3%82%A4%E3%83%B3%E3%82%B5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%BD%E3%83%BC%E3%83%88,%E5%A0%B4%E6%89%80%E3%81%AB%E6%8C%BF%E5%85%A5%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%80%82)


## 例

{:refdef: style="text-align: center;"}
![Fig.2]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort002.svg)
{: refdef}

配列番号 0 から開始します。まずは 0 番目の要素を整列済みとします。
次に 1 番目の要素を整列済みの配列と比較をし、挿入先を探します。挿入先が決まったら、挿入先を空けるために要素を移動します。


{:refdef: style="text-align: center;"}
![Fig.3]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort003.svg)
{: refdef}

移動する要素が２つありました。５を移動してから３を移動させました。これが逆だった場合は５が消えてしまいます。

{:refdef: style="text-align: center;"}
![Fig.4]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort004.svg)
{: refdef}


{:refdef: style="text-align: center;"}
![Fig.5]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort005.svg)
{: refdef}

移動先と元が同じでした。

{:refdef: style="text-align: center;"}
![Fig.6]({{site.baseurl}}/assets/images/insertion_sort/insertion_sort006.svg)
{: refdef}

# コード

```rust
fn insertion_sort(list: &mut [i32]) {

    // 要素番号 0 は整列済みとして 1 から開始
    for i in 1..list.len() {

        // 挿入先を探す
        let mut k = i;
        for j in (0..i).rev() {
            if list[i] < list[j] {
                // 挿入先
                k = j;
            } else if list[k] < list[i] {
                // これ以上左に挿入先となる場所は無いのでループを抜ける
                break;
            }
        }

        // 移動と挿入
        if k != i {

            // 移動先を空けるための移動
            let tmp = list[i];
            for j in (k..i).rev() {
                list[j + 1] = list[j];
            }
            // 挿入
            list[k] = tmp;
        }
    }
}

fn main() {
    let mut list = [5, 3, 1, 2, 6, 4];
    println!("ソート前: {:?}", list);
    insertion_sort(&mut list);
    println!("ソート後: {:?}", list);
}
```

```bash
$ cargo run
ソート前: [5, 3, 1, 2, 6, 4]
ソート後: [1, 2, 3, 4, 5, 6]
```



