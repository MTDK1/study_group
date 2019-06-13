---
layout: post
title:  "Building the Substrate TCR runtime - Step 4"
author: mtdk1
categories: [ Substrate ]
tags: [Substrate, TCR, Tutorial]
description: "Substrate を使った TCR DApp チェーンチュートリアル"
featured: false
hidden: false
toc: true
image: assets/images/TCR.svg
---
原文
Part 1: [Building the Substrate TCR runtime](https://docs.substrate.dev/docs/building-the-substrate-tcr-runtime){:target="_blank"}

目次：[Building a Token Curated Registry DAppChain using Substrate]({{ site.baseurl }}{% post_url 2019-06-11-Building-a-Token-Curated-Registry-DAppChain-using-Substrate %})

## Step 4: Declaring Events

ランタイムモジュール開発の次のステップはイベントを定義することです。一般的に、外部のシステムがブロックチェーンの更新を知るためにはイベントが必要です。もし、ランタイムが正しいイベントを発行しなければユーザーはブロックチェーンに対して大量の on-chain データを要求することになります。

また、Substrate のランタイム関数は成功時に値を返しません。返されるのは空データかエラーメッセージのどちらかです。このため、状態の変化がクライアント、つまりユーザーに伝達されるように必要な情報（パラメータ）を持つイベントを発行することが重要になります。

イベントを使用することでチェーン外に on-chain データのキャッシュを作成することができます。このキャッシュを使用することで、on-chain データを直接参照するよりも高性能な照会や分析を行うことができるようになります。これについては Part 4 で詳しく説明します。

TCR ランタイムモジュールにリストの更新を伝えるイベント（提案、チャレンジ、投票、決議、承認/拒否、報酬請求のイベント）を実装します。これら全てのイベントはリストのライフサイクルと対応するチャレンジにおける論理的なステップであり、これらは外部に伝えられるべきです。

ランタイムモジュールを開発していると以下のようなことを思うことがありませんか？「クライアントまたはユーザは、このランタイムを使用するときどのような種類の更新に興味があるのだろうか？」 理想的には、これらすべての更新がイベントとして伝えられるといいでしょう。

TCRランタイムの場合、```decl_event``` マクロを使用して宣言したイベントがあります。

```rust
decl_event!(
    pub enum Event<T> where 
        AccountId = <T as system::Trait>::AccountId, 
        Balance = <T as token::Trait>::TokenBalance, 
        Hash = <T as system::Trait>::Hash,
    {
      // when a listing is proposed
      Proposed(AccountId, Hash, Balance),
      // when a listing is challenged
      Challenged(AccountId, Hash, u32, Balance),
      // when a challenge is voted on
      Voted(AccountId, u32, Balance),
      // when a challenge is resolved
      Resolved(Hash, u32),
      // when a listing is accepted in the registry
      Accepted(Hash),
      // when a listing is rejected from the registry
      Rejected(Hash),
      // when a vote reward is claimed for a challenge
      Claimed(AccountId, u32),
    }
);
```

ここまででストレージ、genesis config そしてイベントの整理ができたので、ランタイムのビジネスロジックを実装に進むことができます。

## 関連リンク

- [Substrate Runtime Recipes](https://docs.substrate.dev/docs/substrate-runtime-recipes){:target="_blank"}
- Part 1: [Building the Substrate TCR runtime](https://docs.substrate.dev/docs/building-the-substrate-tcr-runtime){:target="_blank"}
- Part 2: [Unit testing the TCR runtime module](https://docs.substrate.dev/docs/unit-testing-the-tcr-runtime-module){:target="_blank"}
- Part 3: [Building a UI for the TCR runtime](https://docs.substrate.dev/docs/building-a-ui-for-the-tcr-runtime){:target="_blank"}
- Part 4: [Building an event based off-chain storage](https://docs.substrate.dev/docs/building-an-event-based-off-chain-storage){:target="_blank"}
- Part 5: [Best Practices](https://docs.substrate.dev/docs/tcr-tutorial-best-practices){:target="_blank"}
- [Creating a Custom Substrate chain](https://docs.substrate.dev/docs/creating-a-custom-substrate-chain){:target="_blank"}
- [Substrate Collectables Tutorial](https://substrate-developer-hub.github.io/substrate-collectables-workshop/#/){:target="_blank"}
- [Explain like I’m 5: Token Curated Registries – Gautam Dhameja](https://www.gautamdhameja.com/token-curated-registries-explain-eli5-a5d4cce0ddbe/){:target="_blank"}
- [Token-Curated Registries 1.0 – Mike Goldin – Medium](https://medium.com/@ilovebagels/token-curated-registries-1-0-61a232f8dac7){:target="_blank"}