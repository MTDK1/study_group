---
layout: post
title:  "Building a Token Curated Registry DAppChain using Substrate"
author: mtdk1
categories: [ Substrate ]
tags: [Substrate, TCR, Tutorial]
description: "Substrate を使った TCR DApp チェーンチュートリアル"
featured: false
hidden: false
toc: false
image: assets/images/TCR.svg
---

これは中級レベルの Substrate フレームワークを使用した DAppsチェーン構築チュートリアルです。
まだ Substrate に慣れていないのであれば基本的な概念をカバーしている下記の初心者用チュートリアルから始めることをお勧めします。

1. [Creating a Custom Substrate chain](https://substrate.readme.io/docs/creating-a-custom-substrate-chain){:target="_blank"} - 速いペースで簡単なチュートリアル
2. [Substrate Collectables Tutorial](https://substrate-developer-hub.github.io/substrate-collectables-workshop/){:target="_blank"} - 詳細なチュートリアルです。おおくの概念を詳細に説明しています。（強くお勧め）


## このチュートリアルでは何をカバーしているか

 Substrate を使って DAppChain を構築しようとする開発者を対象としています。このチュートリアルでは単純な TCR を例として使用していますが TCR については詳しく説明していません。
 もし TCR に興味があれば[ここ](https://www.gautamdhameja.com/token-curated-registries-explain-eli5-a5d4cce0ddbe/){:target="_blank"}で入門できます。TCR 1.0 の全文は[こちら](https://medium.com/@ilovebagels/token-curated-registries-1-0-61a232f8dac7){:target="_blank"} 

### 第1部： TCR Substrateランタイムの構築

- カスタムランタイムモジュールで他のSRMLモジュールを使用する方法
- ランタイムストレージで複合型を使用する方法
- Substrateモジュール用の genesis config の使い方
- ランタイムイベントを使用する方法とタイミング

--
- Step 1: [Setup and prerequisites]({{ site.baseurl }}{% post_url 2019-06-11-Building-the-Substrate-TCR-runtime-step1 %})
- Step 2: [Module trait and types]({{ site.baseurl }}{% post_url 2019-06-11-Building-the-Substrate-TCR-runtime-step1 %})
- Step 3: Declaring the runtime storage
- Step 4: Declaring Events
- Step 5: Module business logic


### 第2部： TCRモジュールの単体テスト

- ランタイムをモックする方法
- genesis config をモックする方法
- ランタイムモジュール関数の単体テストの書き方

### 第3部： Substrate TCRランタイム用のUIの構築

- Polkadot JS API を使用して Substrate ノードに接続する方法
- 複合型をバインドする方法
- チェーン内の状態の更新を購読する方法

### 第4部： クエリと分析のためのイベントベースのオフチェーンストレージの構築

- Substrate ランタイムイベントをリッスンする方法
- チェーン内のデータをチェーン外のストレージと同期させる方法

### 第5部： ベストプラクティス

- ランタイムモジュールにおける経済的脆弱性の回避
- エラー処理 - 決してパニックにならないで！
- いつ状態を更新するか（十分なチェックを行う）


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