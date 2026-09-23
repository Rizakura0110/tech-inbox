# Tech Inbox

記事・タグ・既読活動・JSON backup・metadata取得を所有する製品package。
所有者承認済みのpublic repository `Rizakura0110/tech-inbox`として、rizakura-hontaiへ固定commitのGit submoduleとpnpm workspaceで統合する製品package。npm packageは`private: true`を維持し、npmへ公開しない。独立したWebサイトやCloudflare Workerをここからdeployする構成ではない。

初期sourceは`Rizakura0110/rizakura-hontai`のPhase 32 commit `b6de1e62ad44ee41d52ea3586d0ad923591d0387`にある`packages/tech-inbox`から移した。過去の履歴は基盤repositoryに保持し、このrepositoryには製品の検証済みsnapshotだけを取り込む。基盤の履歴を書き換えず、秘密情報や無関係な製品履歴をコピーしない。

## Entry points

| Export | 用途 |
|---|---|
| `app` | 注入型React画面、製品route、結合test用の製品components/provider |
| `browser` | `TechInboxClient`・共通UIのport型、製品identity |
| `contracts` | 記事/タグ/活動/backup/Queue/fetcherの入出力schema |
| `core` / `core/*` | 状態・日付集計・URL正規化などの純粋処理 |
| `server` | service・domain error・repository/Queue port |
| `schema` | 記事用Drizzle table定義（migrationは持たない） |
| `metadata` | 注入型metadata取得/解析とQueue messageの処理判断 |

`server`・`schema`・`metadata`はbrowserから利用できない。
基盤やDaymarkをimportせず、実DB/Queue、認証付きHTTP、clock、ID生成、共通UI、HTML parserなど必要な能力を基盤が渡す。Cloudflare資格情報や本番設定をこのdirectoryへ追加しない。

## 製品単体の検証

Node.jsは`.node-version`、pnpmは`package.json`の`packageManager`へ固定する。製品repositoryのrootで、固定lockfileを使って次を実行する。

```sh
pnpm install --frozen-lockfile
pnpm check
```

`pnpm check`はformat/lint、source/test型検査、単体test、domain/contracts/metadata/schema coverage、declaration build、依存監査を実行する。UIは注入client/UIの単体testと、基盤側の実HTTP adapterを使う既存結合test/coverage/E2Eで確認する。基盤の生成型やCloudflare資格情報は不要。

`.github/workflows/quality.yml`も同じgateを実行する。CIの権限は`contents: read`のみで、公式checkout actionを完全SHAへ固定し、Node.js/pnpmのアーカイブを固定checksumで検証する。tool、cache、temporary outputはcheckout内へ置き、Cloudflareやnpmへの公開は行わない。

## 基盤との結合検証

基盤repositoryのrootで固定submoduleを取得し、rootの固定lockfileをinstallした後、次を実行する。

```sh
pnpm tech-inbox:check
pnpm tech-inbox:boundaries
pnpm check
```

基盤の`pnpm check`はDaymarkを含む結合版と依存監査まで実行する。製品単体test成功だけではdeployしない。Tech Inboxを先にcommit/pushし、その完全commit SHAを基盤のgitlinkへ記録する。build時にbranchを自動追従せず、組み合わせを検証してから基盤をcommit/pushする。

## 依存と公開物の境界

- 製品の`pnpm-lock.yaml`は単体環境、基盤のlockfileは結合環境を再現する。第三者依存の固定version・integrity、公開後7日gate、peer検査、許可済みinstall scriptだけを使う方針を両方で維持する。
- `rolldown`のoverrideとbuild許可は基盤の既存baselineに合わせたもの。依存を更新する際に、公開直後のversionや未審査scriptの例外を追加しない。
- cache・coverage・dist・node_modules、tool、一時file、資格情報、実際の記事/backupをGitへ含めない。DB migrationと本番環境の設定・操作は基盤だけが管理する。
- `Rizakura0110/tech-inbox`のpublic作成は所有者承認済み。本番反映は基盤側の結合検証と別のdeploy承認後に行い、製品repositoryへのpushで自動deployしない。
