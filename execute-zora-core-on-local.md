# Zora Coreをローカルで動かす

[zora core](https://github.com/ourzora/core) をローカルのチェーン(Ganashe)で動かす方法です。

## clone, build

```sh
git clone git@github.com:ourzora/core.git
cd core
yarn
yarn build
```

## envファイルの作成

ルートディレクトリに`.env.local`を作成。RPC_ENDPOINTとPRIVATE_KEYを設定する。
RPC_ENDPOINTは Ganache のエンドポイント、PRIVATE_KEYはデプロイするアカウントの秘密鍵

```env
RPC_ENDPOINT=http://127.0.0.1:7545
PRIVATE_KEY=b3fc...
```

## 空のaddressesファイルを作成

ChainIdのファイルを作成し中身は空（`{}`）にします。
（GanacheはデフォルトのChainIdは1337）

```sh
echo '{}' > addresses/1337.json
```

## デプロイを実行

```sh
yarn deploy --chainId 1337
```

デプロイが完了すると、address/1337.json が更新されます

```json
{
  "market": "0x1ec...",
  "media": "0x6Be..."
}
```

デプロイしたコントラクトのアドレスを、zdk のコンストラクタに指定する。

```js
const zora = new Zora(
  signer,
  chainId,  // この場合chainIdは無視される
  '0x6Be...',
  '0x1ec...'
);
```
