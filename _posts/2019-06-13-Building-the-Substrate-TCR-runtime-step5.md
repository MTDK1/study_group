---
layout: post
title:  "Building the Substrate TCR runtime - Step 5"
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

## Step 5: Module business logic

最後にビジネスロジックについて説明することには理由は、ここまでの説明でストレージに何を保存しようとしているのか、そして外部に何を伝えようとしているのか（イベント）が明確になっているからです。これはビジネスロジックの最適化に大いに役立ちます。それでは始めましょう。

### Propose a listing

TCR ランタイムモジュールが公開する必要がある最初の機能はリストへの登録提案ができるようにするものです。この関数では、入力としてリストされるアイテム名とデポジットを受け取ります。次に、genesis config とストレージに保存されている TCR パラメータに従って入力された値を検証します。そして、```token```モジュールを使用して関数を実行したユーザの口座残高からデポジットした額を差引ます（ロックします）。最後にリストされるアイテムを ```Listing``` 構造体のインスタンスとしてストレージの```Listings```に格納します。

下記が```Propose```関数の実装です。

```rust
// propose a listing on the registry
// takes the listing name (data) as a byte vector
// takes deposit as stake backing the listing
// checks if the stake is less than minimum deposit needed
fn propose(origin, data: Vec<u8>, #[compact] deposit: T::TokenBalance) -> Result {
  let sender = ensure_signed(origin)?;

  // to avoid byte arrays with unlimited length
  ensure!(data.len() <= 256, "listing data cannot be more than 256 bytes");

  let min_deposit = Self::min_deposit().ok_or("Min deposit not set")?;
  ensure!(deposit >= min_deposit, "deposit should be more than min_deposit");

  // set application expiry for the listing
  // using the `Timestamp` SRML module for getting the block timestamp
  // generating a future timestamp by adding the apply stage length
  let now = <timestamp::Module<T>>::get();
  let apply_stage_len = Self::apply_stage_len().ok_or("Apply stage length not set.")?;
  let app_exp = now.checked_add(&apply_stage_len).ok_or("Overflow when setting application expiry.")?;

  let hashed = <T as system::Trait>::Hashing::hash(&data);

  let listing_id = Self::listing_count();

  // create a new listing instance
  let listing = Listing {
    id: listing_id,
    data,
    deposit,
    owner: sender.clone(),
    whitelisted: false,
    challenge_id: 0,
    application_expiry: app_exp,
  };

  ensure!(!<Listings<T>>::exists(hashed), "Listing already exists");

  // deduct the deposit for application
  <token::Module<T>>::lock(sender.clone(), deposit, hashed.clone())?;

  <ListingCount<T>>::put(listing_id + 1);
  <Listings<T>>::insert(hashed, listing);
  <ListingIndexHash<T>>::insert(listing_id, hashed);

  // let the world know
  // raise the event
  Self::deposit_event(RawEvent::Proposed(sender, hashed.clone(), deposit));
  runtime_io::print("Listing created!");

  Ok(())}
}
```

ストレージを更新する前に、正しい値であることの検証を行わなければなりません。これは、ロジックの途中でエラーが発生し処理が中断してしまった場合、中断する以前におこなったストレージを更新する処理を無かったことにすることができないからです。ストレージを更新する前に細心の注意を払うことは非常に重要なことです。これは[Substrate Collectibles チュートリアル](https://github.com/shawntabrizi/substrate-collectables-workshop/blob/master/2/tracking-all-kitties.md#verify-first-write-last){:target="_blank"}でもよく説明されています。

もし、外部モジュールを使用する場合は使用する関数内で値の検証が行われているかを確認してください。（値の肩章に失敗した場合は処理が中断するため）もし、検証が行われている場合はストレージの更新を行う処理の前にその関数を呼び出す必要があります。ここでは```token```モジュールの```lock```関数を```TCR```モジュールの```propose```関数から呼び出しています。この```token```モジュールの```lock```関数では```propose```関数を呼び出したアカウント(```origin```)の口座残高がdepositよりも多いことを検証しています。そのため TCR モジュールでストレージの更新処理を行う前に、この関数を呼び出さなければなりません。さらに、この```lock```関数でもストレージの更新（ユーザーの資産をロックする処理）が行われます。そのため、この関数を呼び出した以降では失敗する可能性がある処理を絶対に行ってはいけません。

最後に、外部に新しいアイテムがリストに追加する提案が行われたことを伝えるためのイベントを発行します。

### Challenge and Vote
（アイテムをリストに追加する提案に対して拒否をする（チャレンジ）と投票）

上記の ```propose``` 関数と同様に ```challenge``` 関数と ```vote``` 関数も実装します。すべてのチェックと検証をおこなってからストレージを更新するというパターンに従います。

```challenge``` 関数では提案されたリストアイテムが存在していて、まだステージ期間中であるかをチェックします。そして、チャレンジのデポジットが提案されたリストアイテムのデポジット以上であるかチェックします。次に、チャレンジのデポジットをロックし、```Challenge``` 構造体のインスタンスをストレージの ```Challenges``` に格納します。リストアイテムと ```challenge_id``` も更新します。最後に、```Challenged``` イベントを発行します。

```rust
// challenge a listing
// for simplicity, only three checks are being done
//    a. if the listing exists
//    c. if the challenger is not the owner of the listing
//    b. if enough deposit is sent for challenge
fn challenge(origin, listing_id: u32, #[compact] deposit: T::TokenBalance) -> Result {
  let sender = ensure_signed(origin)?;

  ensure!(<ListingIndexHash<T>>::exists(listing_id), "Listing not found.");

  let listing_hash = Self::index_hash(listing_id);
  let listing = Self::listings(listing_hash);

  ensure!(listing.challenge_id == 0, "Listing is already challenged.");
  ensure!(listing.owner != sender, "You cannot challenge your own listing.");
  ensure!(deposit >= listing.deposit, "Not enough deposit to challenge.");

  // get current time
  let now = <timestamp::Module<T>>::get();

  // get commit stage length
  let commit_stage_len = Self::commit_stage_len().ok_or("Commit stage length not set.")?;
  let voting_exp = now.checked_add(&commit_stage_len).ok_or("Overflow when setting voting expiry.")?;

  // check apply stage length not passed
  // ensure that now <= listing.application_expiry
  ensure!(listing.application_expiry > now, "Apply stage length has passed.");

  let challenge = Challenge {
    listing_hash,
    deposit,
    owner: sender.clone(),
    voting_ends: voting_exp,
    resolved: false,
    reward_pool: <T::TokenBalance as As<u64>>::sa(0),
    total_tokens: <T::TokenBalance as As<u64>>::sa(0),
  };

  let poll = Poll {
    listing_hash,
    votes_for: listing.deposit,
    votes_against: deposit,
    passed: false,
  };

  // deduct the deposit for challenge
  <token::Module<T>>::lock(sender.clone(), deposit, listing_hash)?;

  // global poll nonce
  // helps keep the count of challenges and in mapping votes
  let poll_nonce = <PollNonce<T>>::get();
  // add a new challenge and the corresponding poll in the respective collections
  <Challenges<T>>::insert(poll_nonce, challenge);
  <Polls<T>>::insert(poll_nonce, poll);

  // update listing with challenge id
  <Listings<T>>::mutate(listing_hash, |listing| {
    listing.challenge_id = poll_nonce;
  });

  // update the poll nonce
  <PollNonce<T>>::put(poll_nonce + 1);

  // raise the event
  Self::deposit_event(RawEvent::Challenged(sender, listing_hash, poll_nonce, deposit));
  runtime_io::print("Challenge created!");

  Ok(())
}
```

同様に ```vote``` 関数ではリストアイテム、チャレンジが存在していることをチェックします。コミットステージ期間が過ぎているかどうかもチェックします。投票の値（true または false)に基づいて、投票のデポジットを ```poll```インスタンスの ```votes_for``` または ```votes_against``` に追加します。 ```Vote``` 構造体のインスタンスをストレージ ```Votes``` に格納し、```Voted``` イベントを発行します。

```rust
// registers a vote for a particular challenge
// checks if the listing is challenged and
// if the commit stage length has not passed
// to keep it simple, we just store the choice as a bool - true: aye; false: nay
fn vote(origin, challenge_id: u32, value: bool, #[compact] deposit: T::TokenBalance) -> Result {
  let sender = ensure_signed(origin)?;

  // check if listing is challenged
  ensure!(<Challenges<T>>::exists(challenge_id), "Challenge does not exist.");
  let challenge = Self::challenges(challenge_id);
  ensure!(challenge.resolved == false, "Challenge is already resolved.");

  // check commit stage length not passed
  let now = <timestamp::Module<T>>::get();
  ensure!(challenge.voting_ends > now, "Commit stage length has passed.");

  // deduct the deposit for vote
  <token::Module<T>>::lock(sender.clone(), deposit, challenge.listing_hash)?;

  let mut poll_instance = Self::polls(challenge_id);
  // based on vote value, increase the count of votes (for or against)
  match value {
    true => poll_instance.votes_for += deposit,
    false => poll_instance.votes_against += deposit,
  }

  // create a new vote instance with the input params
  let vote_instance = Vote {
    value,
    deposit,
    claimed: false,
  };

  // mutate polls collection to update the poll instance
  <Polls<T>>::mutate(challenge_id, |poll| *poll = poll_instance);

  // insert new vote into votes collection
  <Votes<T>>::insert((challenge_id, sender.clone()), vote_instance);

  // raise the event
  Self::deposit_event(RawEvent::Voted(sender, challenge_id, deposit));
  runtime_io::print("Vote created!");
  Ok(())
}
```

### Resolve

チャレンジされたリストアイテムがステージ期間外になったか、チャレンジされていないリストアイテムがコミットステージ期間外になると、そのリストアイテムについて ```Resolve``` 関数を呼び出すことができます。これはステーキングする必要がなく誰でも呼び出すことができます。

```resolve``` 関数ではいかを含むいくつかの条件をチェックします。

- リストアイテムが存在し、ステージ期間内であるかどうか
- チャレンジが存在し、コミットステージ期間内かどうか
- 投票がホワイトリストに入ること賛成かどうか

これらのチェックに基づいて、```listing.whitelisted``` の値を true または false に設定しリストアイテムのステータスを承認または却下としてストレージを更新します。```Resolved``` 、```Accepted/Rejected``` イベントの発行も行います。

さらに、```Challenge``` インスタンスのトークンと報酬の値も更新します。

```rust
// resolves the status of a listing
fn resolve(_origin, listing_id: u32) -> Result {
  ensure!(<ListingIndexHash<T>>::exists(listing_id), "Listing not found.");

  let listing_hash = Self::index_hash(listing_id);
  let listing = Self::listings(listing_hash);

  let now = <timestamp::Module<T>>::get();
  let challenge;
  let poll;

  // check if listing is challenged
  if listing.challenge_id > 0 {
    // challenge
    challenge = Self::challenges(listing.challenge_id);
    poll = Self::polls(listing.challenge_id);

    // check commit stage length has passed
    ensure!(challenge.voting_ends < now, "Commit stage length has not passed.");
  } else {
    // no challenge
    // check if apply stage length has passed
    ensure!(listing.application_expiry < now, "Apply stage length has not passed.");

    // update listing status
    <Listings<T>>::mutate(listing_hash, |listing| 
    {
      listing.whitelisted = true;
    });

    Self::deposit_event(RawEvent::Accepted(listing_hash));
    return Ok(());
  }

  let mut whitelisted = false;

  // mutate polls collection to update the poll instance
  <Polls<T>>::mutate(listing.challenge_id, |poll| {
    if poll.votes_for >= poll.votes_against {
        poll.passed = true;
        whitelisted = true;
    } else {
        poll.passed = false;
    }
  });

  // update listing status
  <Listings<T>>::mutate(listing_hash, |listing| {
    listing.whitelisted = whitelisted;
    listing.challenge_id = 0;
  });

  // update challenge
  <Challenges<T>>::mutate(listing.challenge_id, |challenge| {
    challenge.resolved = true;
    if whitelisted == true {
      challenge.total_tokens = poll.votes_for;
      challenge.reward_pool = challenge.deposit + poll.votes_against;
    } else {
      challenge.total_tokens = poll.votes_against;
      challenge.reward_pool = listing.deposit + poll.votes_for;
    }
  });

  // raise appropriate event as per whitelisting status
  if whitelisted == true {
    Self::deposit_event(RawEvent::Accepted(listing_hash));
  } else {
    // if rejected, give challenge deposit back to the challenger
    <token::Module<T>>::unlock(challenge.owner, challenge.deposit, listing_hash)?;
    Self::deposit_event(RawEvent::Rejected(listing_hash));
  }

  Self::deposit_event(RawEvent::Resolved(listing_hash, listing.challenge_id));
  Ok(())
}
```

### Claim reward

リストアイテムが resolved となる（リストに登録される）と投票報酬を ```claim_reward``` 関数を使って請求することができます。関数の呼び出し者(origin)が報酬を得る権利があるかどうか、チャレンジに対して投票したかチェックします。そして、チャレンジの投票が終わったかをチェックします。これらのチェックに基づいて、origin に対する報酬額を計算し ```token``` モジュールの ```unlock``` 関数を呼び出します。報酬の請求が繰り返されないように ```Vote``` インスタンスの請求済みステータスを true に設定しストレージを更新します。最後に ```Claimed``` イベントを発行します。

```claim_reward``` 関数の入力は前の関数(resolve)の入力とは異り ```listing_id``` ではなく ```challenge_id``` であることに注意してください。リストアイテムに対する ```challenge_id``` を予め知っている必要があります。

```rust
// claim reward for a vote
fn claim_reward(origin, challenge_id: u32) -> Result {
  let sender = ensure_signed(origin)?;

  // ensure challenge exists and has been resolved
  ensure!(<Challenges<T>>::exists(challenge_id), "Challenge not found.");
  let challenge = Self::challenges(challenge_id);
  ensure!(challenge.resolved == true, "Challenge is not resolved.");

  // get the poll and vote instances
  // reward depends on poll passed status and vote value
  let poll = Self::polls(challenge_id);
  let vote = Self::votes((challenge_id, sender.clone()));

  // ensure vote reward is not already claimed
  ensure!(vote.claimed == false, "Vote reward has already been claimed.");

  // if winning party, calculate reward and transfer
  if poll.passed == vote.value {
        let reward_ratio = challenge.reward_pool.checked_div(&challenge.total_tokens).ok_or("overflow in calculating reward")?;
        let reward = reward_ratio.checked_mul(&vote.deposit).ok_or("overflow in calculating reward")?;
        let total = reward.checked_add(&vote.deposit).ok_or("overflow in calculating reward")?;
        <token::Module<T>>::unlock(sender.clone(), total, challenge.listing_hash)?;

        Self::deposit_event(RawEvent::Claimed(sender.clone(), challenge_id));
    }

    // update vote reward claimed status
    <Votes<T>>::mutate((challenge_id, sender), |vote| vote.claimed = true);

  Ok(())
}
```

```propse```、```challenge```、```vode```、```resolve``` そして ```claim_reward``` 関数を実装したことにより TCR のコア部分が実装できました。必要に応じて、さらに機能を拡張することができます。このチュートリアルで取り上げたのは、あくまでも TCR のサブセットを実装したサンプルです。これは教育目的であり、実際のユースケースを意図して実装されたものではありません。

TCR ランタイムモジュールのコード - このチュートリアルで扱った tcr, token は[ここ](https://github.com/substrate-developer-hub/substrate-tcr/tree/master/runtime/src){:target="_blank"}にあります。

次のパートでは ```reactjs```、```PolkadotJS API``` を使ってフロントエンドからランタイムの関数を呼び出す方法を学びます。
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