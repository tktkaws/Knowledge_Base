# 容量の多いGitの管理方法 — 保存場所と大きなファイルを見直す

## 大きなGitリポジトリとは

- Gitはファイルの変更履歴を管理するツールで、`.git`には履歴や設定などが保存される
- 作業ファイルが18 GB、`.git`が22 GBある場合、履歴と現在のファイルの両方が容量を占める
- cloneは履歴の取得と作業ファイルの展開を行うため、履歴やファイルが大きいと時間がかかる
- Googleドライブ上でエラーが発生していても、原因が同期機能とは限らない — エラーメッセージや同期状態を確認する<sup><a href="#ref-1">[1]</a></sup>

参照

<ol start="1">
  <li id="ref-1"><a href="https://support.google.com/drive/answer/2565956?hl=ja">Google ドライブ ヘルプ — パソコン版ドライブの問題を解決する</a></li>
</ol>

## まず保存場所と容量の内訳を確認する

- 作業用リポジトリをGoogleドライブの同期対象外にあるローカルフォルダに置き、Git対応ホスティングサービスをリモートとして利用する方法を検討する
- Googleドライブの同期とGitの操作が同じファイルに並行して触れる状況を避けられるが、これだけで容量やclone時間が解決するとは限らない
- 移行前に同期完了を確認し、未コミットの変更を含む別のバックアップを作る — 新しい場所で履歴、ブランチ、作業ファイルを確認してから元のコピーを整理する
- `git count-objects -vH`でGitオブジェクトの容量を確認する — `.git`全体の容量と一致するとは限らない<sup><a href="#ref-2">[2]</a></sup>
- `git status`で未追跡ファイルを確認し、生成物や依存パッケージなど、再生成できるものが履歴に含まれていないか調べる
- `.gitignore`は今後の未追跡ファイルを除外するための設定で、すでに追跡中のファイルや過去の履歴を自動では削除しない<sup><a href="#ref-3">[3]</a></sup>

```bash
git status
git count-objects -vH
```

参照

<ol start="2">
  <li id="ref-2"><a href="https://git-scm.com/docs/git-count-objects">Git — git-count-objects</a></li>
  <li id="ref-3"><a href="https://git-scm.com/docs/gitignore">Git — gitignore</a></li>
</ol>

## 保存するものに応じて管理を分ける

- 独立して変更・公開するプロジェクトが混在する場合は、プロジェクトごとにリポジトリを分ける — 必要なものだけcloneできる一方、共通の変更をまとめて扱いにくくなる
- 動画や画像など、履歴も必要な大きいバイナリはGit LFSを検討する — Gitにはポインタを保存し、実体をLFS対応サーバーに保存する<sup><a href="#ref-4">[4]</a></sup>
- Git LFSはサーバーの容量・転送量・料金を確認する — 現在の大きなファイルの取得には時間と容量が必要である
- `git lfs track`を設定するだけでは既存の履歴は変換されない — 過去の履歴も移す場合は別途移行が必要である
- 生成物や一時ファイルは再生成できるならGitに入れず、配布物やバックアップが必要なデータは用途に応じた別のストレージで管理する
- 履歴を分割・書き換える場合はコミットIDが変わり、既存のcloneや共同作業者への影響がある — バックアップを取り、移行を周知して検証する

参照

<ol start="4">
  <li id="ref-4"><a href="https://git-lfs.com/">Git Large File Storage — 公式サイト</a></li>
</ol>

## clone時に取得する範囲を減らす

- 最新の履歴だけで作業できる場合は、`--depth 1`で浅いcloneを作れる — 過去の履歴を参照する作業には追加の取得が必要になる<sup><a href="#ref-5">[5]</a></sup>
- `--filter=blob:none`を使うpartial cloneは、過去のファイル内容を必要になるまで取得しない — チェックアウトする現在のファイルは取得されるため、18 GBの作業ファイルをすべて使う場合はその分が必要になる
- partial cloneにはサーバー側の対応が必要で、未取得の内容を使う際はリモートへの接続が必要になる<sup><a href="#ref-6">[6]</a></sup>
- 一部のディレクトリだけで作業する場合は、partial cloneとsparse checkoutを組み合わせ、展開する範囲を絞る方法もある

```bash
# 最新の履歴だけを取得する
git clone --depth 1 <repository-url>

# ファイル内容の取得を必要になるまで遅らせる
git clone --filter=blob:none <repository-url>
```

参照

<ol start="5">
  <li id="ref-5"><a href="https://git-scm.com/docs/git-clone">Git — git-clone</a></li>
  <li id="ref-6"><a href="https://git-scm.com/docs/partial-clone">Git — Partial Clone</a></li>
</ol>

## まとめ

- Googleドライブ上のエラーは原因を確認し、作業用リポジトリの同期対象外への移動を検討する
- 移行前に未コミットの変更を含むバックアップを作り、移行後に内容を確認する
- `.git`と作業ファイルの容量を分けて調べ、不要な生成物が含まれていないか確認する
- 独立したプロジェクトは分割し、大きなバイナリの履歴にはGit LFSを検討する
- 履歴の書き換えはコミットIDや共同作業者に影響するため、事前に準備する
- 浅いcloneやpartial cloneは取得量を減らせるが、必要な履歴や作業ファイルの取得は残る
