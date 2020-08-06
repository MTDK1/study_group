---
layout: post
title:  "二分木ヒープ"
author: mtdk1
categories: [ プログラミング入門, アルゴリズム ]
tags: [Rust]
description: "プログラミング入門 - 二分木ヒープを実装してみよう"
image: assets/images/3205369_s2.jpg
featured: false
hidden: false
toc: true
mermaid: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

<iframe width="560" height="315" src="https://www.youtube.com/embed/WmSR5WOlwFs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# 構造

二分木ヒープを作るために、まずは木の構造を作ります。今回は構造体は作らずに配列のまま考えていきます。

## 木構造

{:refdef: style="text-align: center;"}
![Fig.1]({{site.baseurl}}/assets/images/binaryheap/bh001.svg)
{: refdef}

二分木はあるノードの子の数は最大２個となる。
上図は大きさ１２、深さ３。

プログラミングで配列の添字は０から始まるが、二分木を配列で構築する場合は添字は１からにする方が理解しやすくなります。

あるノード(n) の子は左の子は(2n)、右の子は(2n+1)になります。
例えば (1) の左の子は (2)、右の子は(3) です。また、(5)の左の子は(10)、右の子は(11) です。
逆に (3) は 3 / 2 = 1 になり親は (1) です。 (12) の親は 12 / 2 = 6 なので親は (6) です。
※ 添字は整数です。rust では整数の割り算では小数点以下は切り捨てになりますので 3 / 2 = 1 になりますが、JavaScript などでは 1.5 となってしまいますので小数点以下を切り捨てる処理が必要です。

{:refdef: style="text-align: center;"}
![Fig.2]({{site.baseurl}}/assets/images/binaryheap/bh002.svg)
{: refdef}

## 制約

次に二分木ヒープの制約を考えます。

- 親ノードの値が子ノードの値よりも小さい
- 親ノードの値が子ノードの値よりも大きい

いずれかの制約を満たすようにします。今回は１つ目の制約を満たすようにします。

## 制約を満たすように値を入れ替える

最後から順に進めていきます。一番下の段は子ノードがないのでチェックする必要がありません。
一番最後のノードである１２に親ノード　６ からチェックし始めます。

親ノードの番号 = 子ノードの番号　÷　２　（小数点以下切り捨て）

ノード１２の親ノード番号は12 / 2 = 6 です。


{:refdef: style="text-align: center;"}
![Fig.3]({{site.baseurl}}/assets/images/binaryheap/bh003.svg)
{: refdef}

上図の青い四角枠で囲ったノードを比較し、必要であれば値を交換します。枠の中のノードで親の値が６、子の値で最小値は１１ですので親の値の方が大きいので交換はしません。



{:refdef: style="text-align: center;"}
![Fig.4]({{site.baseurl}}/assets/images/binaryheap/bh004.svg)
{: refdef}

上図では親が３、二つの子ノードのうち最小値は２です。親の方が大きいので３と２を交換します。
交換後は親が２になり、二つの子ノードよりも値が小さいので制約を満たします。

{:refdef: style="text-align: center;"}
![Fig.5]({{site.baseurl}}/assets/images/binaryheap/bh005.svg)
{: refdef}

上図では親が１、二つの子ノードよりも値が小さいので制約を満たしています。

{:refdef: style="text-align: center;"}
![Fig.6]({{site.baseurl}}/assets/images/binaryheap/bh006.svg)
{: refdef}

上図では親が９、二つの子ノードで最小値は６なので値を入れ替えます。入れ替えると親が６、子が９となり制約を満たします。
入れ替え後の９を親として、その子は１１なので制約を満たしています。

{:refdef: style="text-align: center;"}
![Fig.7]({{site.baseurl}}/assets/images/binaryheap/bh007.svg)
{: refdef}

上図では親が８、この最小値は１なので値を入れ替えます。入れ替え後、８のノードが制約を満たしていません。
親が８、子の最小値は５なので値を入れ替えます。

{:refdef: style="text-align: center;"}
![Fig.8]({{site.baseurl}}/assets/images/binaryheap/bh008.svg)
{: refdef}

上図では親が１２、子の最小値は１なので値を入れ替えます。
入れ替え後、１２のノードが制約を満たしていません。
ノード（４）値１２の子（５）値２を入れ替えます。ノード（５）が１２になりますが、ここでも制約を満たしていません。
ノード（５）値１２の子（１１）値３を入れ替えます。

## 制約を満たすように値を入れ替えた結果


制約を満たす前

{:refdef: style="text-align: center;"}
![Fig.1]({{site.baseurl}}/assets/images/binaryheap/bh001.svg)
{: refdef}

値を入れ替えた後

{:refdef: style="text-align: center;"}
![Fig.9]({{site.baseurl}}/assets/images/binaryheap/bh009.svg)
{: refdef}


ツリーを見ると、全てのノードが制約を満たしていることがわかります。

# コード

```rust
fn min_heapify(list: &mut Vec<u32>, i: usize) {
    
    let left = 2 * i + 1;
    let right = 2 * i + 2;
    let mut smallest = i;

    if left < list.len() && list[left] < list[i] {
        smallest = left
    }
    if right < list.len() && list[right] < list[smallest] {
        smallest = right
    }
    if smallest != i {
        // 値を入れ替える
        let tmp = list[i];
        list[i] = list[smallest];
        list[smallest] = tmp;

        // 入れ替え後のノードについて繰り返す
        min_heapify(list, smallest)
    }
}

fn build_min_heap(list: &mut Vec<u32>) {
    for i in (0..=(list.len()) / 2).rev() {
        min_heapify(list, i)
    }
}
fn main() {
    let mut list = [12, 8, 9, 1, 3, 6, 7, 5, 10, 4, 2, 11].to_vec();
    println!("{:?}", list);
    build_min_heap(&mut list);
    println!("{:?}", list);
}
```

今回は自力でコードを作らず下記ページを参考にさせていただきました。何も参考にせずに作っても同じようになりそうですし、見てしまったので・・・・

- [ヒープをわかりやすく解説してみた. 基本的なデータ構造であるヒープについて、概要、計算量と実装、そして最もシンプルな… \| by Yasufumi TANIGUCHI \| Medium](https://medium.com/@yasufumy/data-structure-heap-ecfd0989e5be)

# ソート

<iframe width="560" height="315" src="https://www.youtube.com/embed/-NlrBdGWraE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```rust

fn min_heapify(list: &mut Vec<u32>, i: usize, length: usize) {
    let left = 2 * i + 1;
    let right = 2 * i + 2;
    let mut smallest = i;

    if left < length && list[left] < list[i] {
        smallest = left
    }
    if right < length && list[right] < list[smallest] {
        smallest = right
    }
    if smallest != i {
        swap(list, i, smallest);
        min_heapify(list, smallest, length)
    }
}

fn build_min_heap(list: &mut Vec<u32>, length: usize) {
    for i in (0..(length) / 2).rev() {
        min_heapify(list, i, length);
    }
}

fn swap(list: &mut Vec<u32>, index_1: usize, index_2: usize) {
    let tmp = list[index_2];
    list[index_2] = list[index_1];
    list[index_1] = tmp;
}

fn heap_sort(list: &mut Vec<u32>) {
    build_min_heap(list, list.len());
    for j in (0..(list.len())).rev() {
        swap(list, 0, j);
        build_min_heap(list, j);
    }
}

fn main() {
    let mut list = [12, 8, 9, 1, 3, 6, 7, 5, 10, 4, 2, 11].to_vec();
    println!("list: {:?}", list);
    heap_sort(&mut list);
    println!("sort: {:?}", list);

    let v: Vec<_> = (1 .. =12).rev().map(u32::from).collect();
    assert_eq!(list, v);
}
```