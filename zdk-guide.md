# Guide

## Minting Cryptomedia

Cryptomedia は、Zora Protocolの基礎となる構成要素です。

Cryptomedia は、インターネット上の誰もが、普遍的にアクセス可能で、個別に所有可能なハイパーメディアを作成するためのメディアです。

ZDK を使用して新しい暗号メディアを作成するには、mint 関数を呼び出す必要がある。

Zoraインスタンス上のmint関数は、2つのパラメータを期待しています。MediaDataとBidSharesです。

### MediaData

`MediaData`の型は以下の4つのフィールドを定義します。
生成するには`generateMetadata`を利用する。

```ts
type MediaData = {
  tokenURI: string
  metadataURI: string
  contentHash: BytesLike
  metadataHash: BytesLike
}
```

#### tokenURI

CryptomediaのコンテンツがホストされているURI。
これはインターネット上の任意のストレージプロバイダーにリンクする。  
分散型ストレージプロバイダーの例として

- ipfs
- arweave
- storj
- sia

tokenURIのプレフィックスは必ず `https://` を付けなければなりません。

#### metadataURI

CryptomediaのメタデータがホストされているURI。
tokenURIと同じように任意のストレージプロバイダーにリンクする必要がある。

#### contentHash

Cryptomedia が表すコンテンツの sha256 ハッシュです。一度ブロックチェーンに書き込まれたハッシュは変更できないため、このハッシュが正しいことが必須です。このハッシュを生成するには、utilsで定義されている`sha256 utils`を使用する。

#### metadataHash

Cryptomedia のメタデータの sha256 ハッシュです。一度ブロックチェーンに書き込まれたハッシュは変更できないため、このハッシュが正しいことが必須です。このハッシュを生成するには、utilsで定義されている `sha256 utils` を使用します。

```ts
import { constructMediaData, sha256FromBuffer, generateMetadata } from '@zoralabs/zdk'

const metadataJSON = generateMetadata('zora-20210101', {
  description: '',
  mimeType: 'text/plain',
  name: '',
  version: 'zora-20210101',
})

const contentHash = sha256FromBuffer(Buffer.from('Ours Truly,'))
const metadataHash = sha256FromBuffer(Buffer.from(metadataJSON))
const mediaData = constructMediaData(
  'https://ipfs.io/ipfs/bafybeifyqibqlheu7ij7fwdex4y2pw2wo7eaw2z6lec5zhbxu3cvxul6h4',
  'https://ipfs.io/ipfs/bafybeifpxcq2hhbzuy2ich3duh7cjk4zk4czjl6ufbpmxep247ugwzsny4',
  contentHash,
  metadataHash
)
```

### BidShares

`BidShares`の型は以下の3つのフィールドを定義します。
生成するには`constructBidShares`を利用する。

```ts
type DecimalValue = { value: BigNumber }

type BidShares = {
  owner: DecimalValue
  prevOwner: DecimalValue
  creator: DecimalValue
}
```

各フィールドは、Cryptomedia の各ステークホルダーが次回の入札額における割合を表す。
Mint時、各ステークホルダーの入札シェア（クリエーター、オーナー、前オーナー）の合計が100であるひつようがある。


#### creator

Cryptomedia の承認された入札から、作成者が取得する不変の恒久的な持分(%)。

#### owner

現在の所有者が、Cryptomedia の次回の入札で得られる持分(%)。

#### prevOwner

前の所有者は、Cryptomedia の作品の次回の入札で得られる持分(%)を取得します。

`constructBidShares`メソッドでは、精度を単純化するために、JSの数値を受け入れ、10進数4位までで丸めた Ether の BigDecimal型に変換される。その際に合計値が100にならない場合はエラーとなる。

例えば `24.44445` は丸められて `24.4444` となるので合計が100にならない場合がある。

## Bidding on Cryptomedia

Zoraプロトコルのユーザーが所有したい Cryptomedia を見つけた場合、入札を行うことができます。

入札は、入札者が選択したERC-20で行われます。

入札を作成するには `constructBid` ユーティリティ関数を使用し、Zora インスタンスで `setBid` メソッドを呼び出します。

**Note:** Cryptomedia の一部に入札を行うには、Zora Market コントラクトに資金を入金する必要があります。
入札を成功させるためには、Zora Market コントラクトを承認して資金を送金する必要があります。これには `approveERC20` メソッドを使用することができます。
