---
layout: post
title:  "プログラミング入門"
author: mtdk1
categories: [ プログラミング入門 ]
tags: [アルゴリズム, Python, プログラミング、フローチャート]
image: assets/images/algorithm.png
description: "基本的なアルゴリズムを学習を通してプログラミングの学習をする"
featured: false
hidden: false
toc: true
---

目次：[プログラミング学習]({{ site.baseurl }}{% post_url 2020-07-23-list_programming %})

---

基本的なアルゴリズムの学習を通して、フローチャートやプログラミングの学習をします。

フローチャートの書き方やプログラミング言語の基本仕様の学習が目的ですので、ここで紹介するアルゴリズムを自分の手で実装してみてください。

この記事は BDevUx 勉強会の内容を整理して追記していきます。

---
# Hello World!!（１）

プログラミング入門と言ったら「Hello World!!」です。 環境構築ができたら Hello World と画面に表示してみましょう。

Python
```python
print('Hello World!!')
```

JavaScript
```javascript
console.log("Hello World!!")
```

Rust
```rust
fn main() {
  println!("Hello World!!");
}
```

Python や JavaScript であれば簡単だ！ という人は多いでしょう。
Rust の場合は実行ファイルを作成する必要があります。ちょっとハードルが高くなりますね。
Google で「（言語名） Hello World」で検索してみてください。

# Hello World!!（２）

次は配列に Hello World!! を設定して for 文を使って表示してみます。

1. 配列を初期化する
2. 一文字ずつコンソールに出力する

Python
```python
list = ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd', '!', '!', '\n']
for item in list:
    print(item, end='')
```

```bash
$ python3 helloworld.py
Hello World!!
```


JavaScript (Node.js)
```javascript
let list = ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd', '!', '!', '\n']
for (let i in list) {
    process.stdout.write(list[i])
}

list.forEach(item => {
    process.stdout.write(item)
});
```

```bash
$ node helloworld.js
Hello World!!
Hello World!!
```

Rust
```rust
fn main() {
    let list = ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd', '!', '!', '\n'];
	for item in list.iter() {
		print!("{}", item);
	}
}
```

```bash
$ rustc helloworld.rs
$ ./helloworld
Hello World!!
```

言語によって書き方が違いますが、やっていることは同じことです。
次回からアルゴリズム、フローチャート、実装について書いていきます。

時間があれば Kotlin, Swift のコードも紹介していきます。
