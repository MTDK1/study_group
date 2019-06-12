---
layout: post
title:  "Building the Substrate TCR runtime - Step 3"
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

## Step 3: Declaring the runtime storage

まず最初にやることは、TCR ランタイム用のストレージを定義することです。

DApp または DAppChain を構築する際に最も重要なことは、on-chain に何を保存し、何を保存しないのかを
決めることです。データの競合を解決するために必要なデータのみを保存することをお勧めします。それ以外のデータは off-chain（チェーン外）に保存するべきです。ユーザが支払ったリソースとネットワークから提供されたリソースを一致させるための経済的セキュリティにとってストレージを正しく定義することが重要です。ストレージアイテムが大きくなったり、対処することが面倒なものになってしまうとトランザクションが複雑になり経済的な DoS 攻撃を受ける可能性があります。

TCR ランタイムにおいて、チェーン上に保存されるデータ量が最小限となるように何ができるか確認しましょう。TCRの基本的なキュレーション操作を実行するには、リスト（レジストリ）、チャレンジ、投票（リストIDや投票結果などの情報）と票（賛成、反対、デポジットなどの情報）のコレクションを保存します。TCRパラメータもストレージに保存する必要があります。

### Data structures for on-chain TCR data

まず、リスト、チャレンジ、投票のデータ構造を```Struct```を使用して定義します。

```rust
#[derive(Encode, Decode, Default, Clone, PartialEq)]
#[cfg_attr(feature = "std", derive(Debug))]
pub struct Listing<U, V, W> {
  id: u32,
  data: Vec<u8>,
  deposit: U,
  owner: V,
  application_expiry: W,
  whitelisted: bool,
  challenge_id: u32,
}

#[derive(Encode, Decode, Default, Clone, PartialEq)]
#[cfg_attr(feature = "std", derive(Debug))]
pub struct Challenge<T, U, V, W> {
  listing_hash: T,
  deposit: U,
  owner: V,
  voting_ends: W,
  resolved: bool,
  reward_pool: U,
  total_tokens: U
}

#[derive(Encode, Decode, Default, Clone, PartialEq)]
#[cfg_attr(feature = "std", derive(Debug))]
pub struct Vote<U> {
  value: bool,
  deposit: U,
  claimed: bool,
}

#[derive(Encode, Decode, Default, Clone, PartialEq)]
#[cfg_attr(feature = "std", derive(Debug))]
pub struct Poll<T, U> {
  listing_hash: T,
  votes_for: U,
  votes_against: U,
  passed: bool,
}
```
これらはジェネリック構造体なので、型パラメータに実際の型を指定して利用します。つぎのサブセクションでは、これらの構造体を初期化する方法について説明します。```Moment``` 型は ```timestamp``` モジュール、```TokenBalance``` 型は ```token``` モジュールを利用しています。


### Storage declaration using the decl_storage macro

TCR ランタイムでの```decl_storage```マクロの使い方です。各コメントはストレージアイテムの説明です。

```rust
decl_storage! {
 trait Store for Module<T: Trait> as Tcr {

    // genesis config にある Owner を保存
    Owner get(owner) config(): T::AccountId;

    // TCR parameter - デポジット額の最小値
    MinDeposit get(min_deposit) config(): Option<T::TokenBalance>;

    // TCR parameter - ステージ長 - リストが受け入れられるまでのチャレンジ期間
    ApplyStageLen get(apply_stage_len) config(): Option<T::Moment>;

    // TCR parameter - コミットステージ長 - チャレンジが終わるまでの投票期間
    CommitStageLen get(commit_stage_len) config(): Option<T::Moment>;

    // the TCR - リスト
    Listings get(listings): map T::Hash => Listing<T::TokenBalance, T::AccountId, T::Moment>;

    // リストのクエリ作成、メンテナンスを容易にするためのインデックスとハッシュ
    ListingCount get(listing_count): u32;
    ListingIndexHash get(index_hash): map u32 => T::Hash;

    // 投票（poll)カウントのためのグローバル nonce
    PollNonce get(poll_nonce) config(): u32;

    // チャレンジ
    Challenges get(challenges): map u32 => Challenge<T::Hash, T::TokenBalance, T::AccountId, T::Moment>;

    // 投票（polls）
    Polls get(polls): map u32 => Poll<T::Hash, T::TokenBalance>;

    // 投票（votes）
    // poo id と Account ID を投票（votes) とマッピングする
    // poll と vote は 1:n（1対多） の関係
    Votes get(votes): map (u32, T::AccountId) => Vote<T::TokenBalance>;

    }
}
```

見てのとおり、前述のストレージアイテムに加えて、データのクエリを簡単にするために追加のアイテム（```ListingCount```と```ListingIndexHash```）があります。これらは完全にオプションであり TCR ランタイムのコア機能はそれらを必要としているわではありません。

### Using the genesis config

genesis config はストレージの初期値を定義するものです。これはチェーン起動後のトランザクションに必要なパラメータを指定する方法として役に立ちます。例えば、TCR パラメータはリストを作成する前に必要になります。

Substrateでは、config() をストレージアイテムの宣言に追加することで、そのアイテムの値を genesis config にある値で設定できます。TCRランタイムでは、初期値を設定するものとして下記のものがあります。それぞれのストレージアイテムの宣言に config() が書かれています。

```rust
// stores the owner in the genesis config
Owner get(owner) config(): T::AccountId;

// TCR parameter - minimum deposit
MinDeposit get(min_deposit) config(): Option<T::TokenBalance>;

// TCR parameter - apply stage length - deadline for challenging before a listing gets accepted
ApplyStageLen get(apply_stage_len) config(): Option<T::Moment>;

// TCR parameter - commit stage length - deadline for voting before a challenge gets resolved
CommitStageLen get(commit_stage_len) config(): Option<T::Moment>;
```

ストレージアイテムの宣言に config() を書くと、genesis config の値を使用できるようになります。最初のブロックの前のストレージの値に genesis config の値を使用するためには、genesis config に値を設定する必要もあります。

genesis config に値を設定するためには、```chain_spec.rs``` を編集します。

まず、テンプレートランタイムのインポート(use)に型名を追加する必要があります。以下のコードスニペットではTCRランタイムモジュールの genesis config である```TcrConfig```をインポート(use)に追加しました。

```rust
use node_template_runtime::{
    AccountId, GenesisConfig, ConsensusConfig, TimestampConfig, BalancesConfig,
    SudoConfig, IndicesConfig, TcrConfig
};
```

次に、同じ```chain_spec.rs```ファイルにある```testnet_genesis```関数にセクションを追加します。TCRランタイムに関する genesis config パラメータは下記のようになります。


```rust
tcr: Some(TcrConfig {
    // owner account id
    owner: ed25519::Pair::from_seed(b"Alice ").public().0.into(),

    // min deposit for proposals
    min_deposit: 100,

    // challenge time limit - for testing its set to 2 mins (120 sec)
    apply_stage_len: 120,

    // voting time limit - for testing its set to 4 mins (240 sec)
    commit_stage_len: 240,

    // initial poll/challenge set to 1
    // to avoid 0 values
    poll_nonce: 1,
})
```

ここまでで ```decl_storage```マクロ内の config() が書かれたランタイムストレージアイテムすべての初期値を設定することができました。たとえば、デポジット額の最小値(min_deposit)は 100 で、この値はチェーンの起動直後から使用されるため、作成される新しいリストのデポジットはこの値で検証されます。

[ここ](https://github.com/substrate-developer-hub/substrate-tcr/blob/master/src/chain_spec.rs){:target="_blank"} で ```chain_spec.rs``` を確認することができます。

これでストレージの宣言とセットアップに必要な作業は完了しました。


