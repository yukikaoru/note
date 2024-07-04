---
title: "Slack APIのメッセージのBlock Kitによるレイアウト"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["slack"]
published: true
---

# Slack APIのメッセージのBlock Kitによるレイアウト

Slack APIのメッセージのレイアウト方法を調べると、まだまだ古い仕様での解説のものがたくさん出てくる状態ですので、Block Kitによるレイアウト方法のHow Toと困ったことのメモです。

仕様などの情報は[公式ドキュメント](https://api.slack.com/surfaces)を見ます。
[古い仕様からの移行方法](https://api.slack.com/messaging/attachments-to-blocks)も参考にしたアプリなどからレイアウトを起こす場合に必要になったりもするでしょう。


## カラーバーは使わないようにする

メッセージの横に色付きの棒が置かれているものをCIやアラート系アプリで良く見ます。
![](/images/slack-api-message-layout/1.png)

これはBlock Kitでは添付ファイル系で利用するAttachmentでしか利用できません。
Attachmentは通常のMessageとは違い、長いコンテンツは折り畳まれて表示されます。
それでも構わない場合はAttachmentを利用しても構わないとは思いますが、Slack自身も[例外措置としての利用を説明しているだけで利用には消極的](https://api.slack.com/messaging/attachments-to-blocks#direct_equivalents)です。
> There is one exception, and that's the color parameter, which currently does not have a block alternative. If you are strongly attached (🎺) to the color bar, use the blocks parameter within an attachment.

Messageとは異なる表示になるかもしれませんので利用するのは避けたほうが無難でしょう。（しかしながら2019年にリリースされてもう何年経つのか


## マークダウン表記で日本語ドメイン形式の文字列がPunycodeに変換されてしまう

`mrkdwn`ブロックを利用した時に、日本語ドメイン形式の文字列がPunycodeに変換されてしまいます。
[サンプルコード](https://app.slack.com/block-kit-builder/T02SS7546#%7B%22blocks%22:%5B%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22%E3%81%8A%E5%90%8D%E5%89%8D.com%22%7D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22%60%E3%81%8A%E5%90%8D%E5%89%8D.com%60%22%7D%7D%5D%7D)
![](/images/slack-api-message-layout/2.png)

これを避けるには`verbatim`属性をtrueに設定します。
[サンプルコード](https://app.slack.com/block-kit-builder/T02SS7546#%7B%22blocks%22:%5B%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22%E3%81%8A%E5%90%8D%E5%89%8D.com%22,%22verbatim%22:true%7D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22mrkdwn%22,%22text%22:%22%60%E3%81%8A%E5%90%8D%E5%89%8D.com%60%22,%22verbatim%22:true%7D%7D%5D%7D)
![](/images/slack-api-message-layout/3.png)
