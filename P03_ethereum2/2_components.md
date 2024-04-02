- [各構成要素が保持するデータ](#各構成要素が保持するデータ)
- [まとめ](#まとめ)


# 各構成要素が保持するデータ

各構成要素が保持するデータは、以前のEthereumと同様である

- [EOA (Externally Owned Account)](../P02_ethereum/2_components.md#eoa-externally-owned-account)
- [CA (Contract Account)](../P02_ethereum/2_components.md#ca-contract-account)
- [Message Call トランザクションのデータ構造](../P02_ethereum/2_components.md#message-call-トランザクションのデータ構造)
- [Ethereumの手数料 (gas)](../P02_ethereum/2_components.md#ethereumの手数料-gas)
- [Message Call トランザクションのレシート](../P02_ethereum/2_components.md#message-call-トランザクションのレシート)
- [Contract Creation トランザクションのデータ構造](../P02_ethereum/2_components.md#contract-creation-トランザクションのデータ構造)
- [Contract Creation トランザクションのレシート](../P02_ethereum/2_components.md#contract-creation-トランザクションのレシート)
- [データ構造を実際に確認してみましょう](../P02_ethereum/2_components.md#データ構造を実際に確認してみましょう)


バリデータが保持するデータ

秘密鍵: バリデーターは、自身のトランザクションを署名するための秘密鍵を保持します。これは、attestationやブロック提案のための署名に使われ、他の人と共有されるべきではありません。

クライアントソフトウェアの設定: バリデーターは、自分のクライアントソフトウェアの設定を管理し、パフォーマンスの最適化、ネットワーク設定、ログ記録などを調整します。

Beacon Chainのデータ: Beacon Chainに関する重要な情報、特に現在の状態や最近のブロックに関するデータは、バリデーターが迅速にアクセスできるようにローカルにキャッシュされることがあります。

スラッシング保護データ: 二重署名や他のスラッシングイベントを防ぐため、バリデーターは自身の過去の投票記録などのデータを保持することがあります。これにより、不正な行動を避け、ペナルティを受けるリスクを減らすことができます。

バックアップデータ: 安全性を高めるため、バリデーターは重要なデータ（例えば、秘密鍵、設定ファイル、スラッシング保護データなど）のバックアップを別の安全な場所に保管することが推奨されます。


# まとめ
- 各構成要素が保持するデータは、以前のEthereumと同様である
- しかしこれらの構成要素がどう連動するかについて説明する次の資料からは、内容が大きく異なりはじめる