---
layout: post
title:  "挿入ソート"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust, Swift]
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

> まず0番目と1番目の要素を比較し、順番が逆であれば入れ換える。次に、2番目の要素が1番目までの要素より小さい場合、正しい順に並ぶように「挿入」する（配列の場合、前の要素を後ろに一つずつずらす）。この操作で、2番目までのデータが整列済みとなる（ただし、さらにデータが挿入される可能性があるので確定ではない）。このあと、3番目以降の要素について、整列済みデータとの比較と適切な位置への挿入を繰り返す。<br>
> [挿入ソート - Wikipedia](https://ja.wikipedia.org/wiki/%E6%8C%BF%E5%85%A5%E3%82%BD%E3%83%BC%E3%83%88)


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

# コード(rust)

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



# コード(Swift)

```swift
public func insertionSort(_ array: inout [Int]) {
     // 1個目の要素を整列済みとし、２個目の要素（要素番号1）から開始
       for i in 1..<array.count {
           var k = i;
           
           // 要素 i より左にある要素と比較、挿入先を探す
           for j in (0..<i).reversed() {
               if array[i] < array[j] {
                   k = j;
               } else if array[k] < array[i] {
                   break;
               }
           }
           
           if k != i {
               // 配列から要素iを削除
               // 削除した要素をkに挿入
               array.insert(array.remove(at: i), at: k)
           }
       }
}
```



[変更履歴]
- 2020/09/03: Swift コードを追加しました