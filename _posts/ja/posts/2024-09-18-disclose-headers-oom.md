---
title: ヘッダースパムによるメモリDoSの開示
name: blog-disclose-headers-oom
id: ja-blog-disclose-headers-oom
lang: ja
permalink: /ja/2024/09/18/disclose-headers-oom/
type: advisory
layout: post

## If this is a new post, reset this counter to 1.
version: 1

## Only true if release announcement or security annoucement. English posts only
announcement: 0

excerpt: >
  攻撃者は、低難易度のヘッダーチェーンでBitcoin Coreノードをスパムし、リモートでノードをクラッシュさせる可能性がありました。
---

Bitcoin Core v24.0.1より前のバージョンでは、攻撃者は低難易度のヘッダーチェーンを使用してノードをスパムし、
ピアをリモートでクラッシュさせる可能性がありました。

この問題の重大度は**高**です。

## 詳細 {#details}

Bitcoin Coreは、ブロックチェーンのヘッダーをメモリに格納します。
このため、たとえ難易度が低くても、非常に長いヘッダーチェーンをダウンロードし格納することで、DoSを受けやすくなります。
重要なのは、一度作成された攻撃用のチェーンは、ネットワーク上の任意のノードをクラッシュさせるために再利用できるということです。

これを利用してノードを攻撃する可能性は以前から知られており、チェックポイントシステムが残っていた主な理由でもありました。
攻撃者に最後のチェックポイントから攻撃を開始させることで、ジェネシスブロックから開始するよりもはるかにコストがかかるからです。
しかし、時間の経過と共にハッシュレートコストが低下すると、この緩和策の効果も低下しました。

この攻撃は、2019年1月にDavid Jaensonによって発見され、Bitcoin Coreプロジェクトに報告されました。
彼は、実用的な緩和策として、新しいチェックポイントの導入を提案しました。しかし、
1. これでは、チェックポイントのブロックを受信するまで、IBDを実行しているノードは保護されないままです。
2. これは、エコシステムが新しいチェックポイントを備えた更新済みのソフトウェアを定期的に採用することに依存しており、
   Bitcoin Coreのコントリビューターは長い間この慣行に不快感を抱いていました。

その後、2019年10月にBraydon Fullerが[「Chain width expansion」に関する記事](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-October/017354.html)を
bitcoin-devメーリングリストに投稿したことで、この問題の注目が高まりました。
彼は以前、責任をもってBitcoin Coreのセキュリティリストに報告していました。
提案されたアプローチは、並列チェーンの数を制限する際のネットワークの収束に関する懸念から
Bitcoin Coreでは採用されませんでした。

当時、巨大な低難易度ヘッダーチェーンを作成するための計算コストは、先端の1ブロックのマイニングの約32.28%に相当しました。
当時のマイニング報酬は、約12.77 BTCだったため、これは約4.12 BTCのコストです。

2022年2月までに、攻撃のコストはさらに下がり、ブロックのマイニングコストの約14.73%となり、代替ソリューションの調査が促進されました。
未対策の場合、現在（2024年9月）のコストはブロックの4.44%に過ぎません。
それぞれの時点のブロック報酬を考慮すると、それぞれ約1.07 BTCと0.14 BTCのコストに相当します。

このDoSに対する保護は、Bitcoin CoreのPR[#25717](https://github.com/bitcoin/bitcoin/pull/25717)で実装され、
それによってノードは提示されたチェーンに十分な作業があることを検証してから、チェーンを格納することをコミットします。
これにより、Bitcoin Coreは既知の攻撃から保護するためにチェックポイントに依存する必要がなくなりました。

## 貢献 {#attribution}

攻撃を独自に再発見し、そのコストを見積もり、修正を提案してくれたDavid JaensonとBraydon Fullerに感謝します。

満足のいく修正を調査し、実装してくれたSuhas DaftuarとPieter Wuilleに感謝します。

## タイムライン {#timeline}

* 2010-07-17 - チェックポイントを導入したBitcoin 0.3.2がリリースされる。これは特に低難易度のブロックスパムから保護します。
* 2011-11-21 - Bitcoin 0.5.0がリリースされる。これにより、最後のチェックポイントより前のブロックのスクリプトの検証がスキップされます。
  これにより、チェックポイントの役割がセキュリティ上さらに重要になります。
* 2014-04-09 - ブロック295000がマイニングされ、これが最後のBitcoin Coreのチェックポイントになる。
  ハッシュレートコストが減少するにつれて、チェックポイントによりブロックスパムから保護する機能は、この時点から弱まりはじめます。
* 2015-02-16 - ヘッダーファースト同期を実装したBitcoin Core 0.10.0がリリースされる。
  これにより低難易度のブロックスパム攻撃がブロックヘッダースパム攻撃に弱まります。
* 2017-03-08 - Bitcoin Core 0.14.0がリリースされる。これによりスクリプトの検証のスキップがチェックポイントから切り離され、
  チェックポイントはブロックヘッダースパムの防止のみに関連するようになりました。
* 2019-01-28 - David Jaensonがこの問題をBitcoin Coreのセキュリティメーリングリストに報告。
* 2019-09-18 - Braydon Fullerが「[Bitcoin Chain Width Expansion Denial-of-Service
  Attacks](https://bcoin.io/papers/bitcoin-chain-expansion.pdf)」というタイトルの論文をBitcoin Coreのセキュリティリストにメールで送信。
  この論文では、ブロックとブロックヘッダースパムの危険性、コスト分析および提案されたソリューションについて説明しています。
* 2019-09-26 - Suhas DaftuarがBraydon Fullerにこれは既知の問題であると返信し、
  bitcoin-devメーリングリストにレポートを投稿するよう依頼。
* 2019-10-04 - Braydon Fullerが論文をbitcoin-devメーリングリストに送信。
* 2019-10-31 - 上記の出来事を受けて、Suhas Daftuarが、このトピックに関するより多くの議論が起こることを期待して、
  この状況を改善するために彼が取り組んだ初期の（ただし現実的ではない）概念実証をPR[#17332](https://github.com/bitcoin/bitcoin/pull/17332)で公開。
* 2022-02    - Suhas DaftuarとPieter Wuilleがこの問題について議論し、
  この攻撃コストが実際に非常に低くなったため即時対応が必要で、公に話すことは避ける必要があると推定しました。
* 2022-06-22 - Suhas Daftuarが修正に向けた準備作業としてPR [#25454](https://github.com/bitcoin/bitcoin/pull/25454)を公開。
* 2022-06-22 - Suhas Daftuarは、攻撃の詳細およびそのコストと、Pieter Wuilleと彼が取り組んできた修正について、
  長期のコントリビューターのグループにメッセージを送信。
* 2022-07-26 - Suhas DaftuarがPieter Wuilleと共同で修正を実装したPR [#25717](https://github.com/bitcoin/bitcoin/pull/25717)を公開。
* 2022-08-30 - PR #25717 がマージされる。
* 2022-10-21 - Niklas GöggeのPR [#26355](https://github.com/bitcoin/bitcoin/pull/26355)がマージされ、
  PR #25717で導入されたヘッダーの事前同期手順のバグが修正される。これがなければ、
  ブロックヘッダーのスパムが引き続き可能でした。このバグの発見と、未発見のバグの可能性が、
  古いチェックポイントがまだ完全に削除されない理由です。
* 2022-12-12 - Bitcoin Core 24.0.1が修正と共にリリースされる。
* 2023-12-07 - 脆弱性のある最後のバージョンのBitcoin Core（23.2）がEOLになる。
* 2024-09-18 - 公開。

{% include references.md %}