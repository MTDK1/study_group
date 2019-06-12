---
layout: post
title:  "Building the Substrate TCR runtime - Step 1, 2"
author: mtdk1
categories: [ Substrate ]
tags: [Substrate, TCR, Tutorial]
description: "Substrate を使った TCR DApp チェーンチュートリアル"
featured: false
hidden: false
toc: true
image: assets/images/TCR.svg
---

目次：[Building a Token Curated Registry DAppChain using Substrate]({{ site.baseurl }}{% post_url 2019-06-11-Building-a-Token-Curated-Registry-DAppChain-using-Substrate %})

このパートでは TCR ランタイムに必要な Substrate ランタイムモジュールの実装を行います。

## Step 1: Setup and prerequisites(設定と前提条件)

このチュートリアルでは ```substrate-node-new``` スクリプトを使用して作成された Substrate ノードに TCR ランタイムモジュールを直接追加していきます。このチュートリアルは Substrate を使って DAppChain を作成するプロセス全体を説明することを目的としていません。"getting started"（はじめに）やその他の概念は網羅されていません。これらの概念を理解したければ、先に進む前にチュートリアル「[Substrate Collectables Tutorial](https://substrate-developer-hub.github.io/substrate-collectables-workshop/#/){:target="_blank"}」を実行することを強くお勧めします。

それでは新しい Substrate ノードを使ってチュートリアルをはじめましょう。```substrate-node-new``` スクリプトを使用して新しい Substrate ノードを作成する手順については、チュートリアル「[Using the Substrate node and module setup scripts](https://docs.substrate.dev/docs/substrate-up-scripts){:target="_blank"}」を実行することをお勧めします。

## Step 2: Module trait and types (モジュール trait と型)

Substrate ランタイムモジュール（カスタムモジュール）を構築する最初のステップは、どの SRML モジュールを使用するかを定義することです。Substrate コードには多くの SRML が含まれています。カスタムモジュールに必要な機能が含まれているのであれば可能な限り、それら既存の SRML を使用することをお勧めします。既存のモジュールを使用するためには、使用したいモジュールをインポートして、カスタムモジュールの trait で指定します。

例えば、このランタイムでは TCR パラメータのタイムスタンプを比較する機能、つまりステージ長とコミットステージ長を適用する機能が必要です。この機能を実現するために ```timestamp``` [SRML](https://github.com/paritytech/substrate/tree/master/srml/timestamp){:target="_blank"} を使用します。

TCR モジュールの設定 trait は下記のように宣言します。

```rust
pub trait Trait: timestamp::Trait + token::Trait {
  type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
}
```

このスニペットでは ```timestamp``` を継承した TCR モジュール の 設定 trait があります。さらに、```token``` も継承しています。これらのモジュールを使用するためには先にインポートする必要があります。次のスニペットでは SRML から ```timestamp``` をインポートし、ローカルクレートから ```token``` をインポートしています。

```rust
use {system::ensure_signed, timestamp};
use crate::token;
```

```token``` は、TCR キュレーション機能のトークン機能をサポートするために必要なもう1つのカスタムモジュールです。このチュートリアルでは ```token``` は自明であるため、実装の詳細は説明しません。この ```token``` は ERC20 インターフェースといくつかの追加機能（ロックとロック解除）を実装しています。

コード全体は [TCR サンプルコード](https://github.com/substrate-developer-hub/substrate-tcr){:target="_blank"}を参照してください。
- [tcr.rs](https://github.com/substrate-developer-hub/substrate-tcr/blob/master/runtime/src/tcr.rs){:target="_blank"}
- [token.rs](https://github.com/parity-samples/substrate-tcr/blob/master/runtime/src/token.rs){:target="_blank"}

tcr.rs と token.rs の参照を lib.rs に追加し、それらを ```construct_runtime!``` マクロに追加することを忘れないでください。
