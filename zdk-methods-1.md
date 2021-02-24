# Docs

## Zora

ZoraクラスはZoraプロトコルの指定のインスタンスを読み書きする。

### コンストラクタ

```ts
constructor(
    signerOrProvider: Signer | Provider,
    chainId: number,
    mediaAddress?: string,
    marketAddress?: string
  )
```

以下の2つは必須

* signerOrProvider
* chainID

#### signerOrProvider

コンストラクタは signerOrProvider を使用して、Zoraインスタンスが `readOnly` か `readAndWrite` のどちらに対応しているかを判断します。

`Signer`の値が渡されたとき、Zoreインスタンスは read, writeの両方を呼び出すことができる。

`Provider`の値が渡されたは、readOnlyのメソッドを呼び出すようになる。

#### chainID

コンストラクタは chainID を使用して公式にサポートされているZoraプロトコルのアドレスを検索し、そのプロトコルに接続されたZoraインスタンスを返します。  
Mainnet の場合:1、Rinkebyの場合:4 （しかない？)  
https://github.com/ourzora/core/tree/master/addresses

#### mediaAddress, marketAddress

ローカルのブロックチェーン上で開発する場合、このオプションにコントラクトのアドレスを指定することによってコントラクトの指定先を上書きすることができる。
両方設定するか両方nullである必要がある。
アドレスを設定した場合は、chainIDは無視される。

### 利用例

```ts
import { Zora } from '@zoralabs/zdk'
import { Wallet } from 'ethers'

const wallet = Wallet.createRandom()
const zora = new Zora(wallet, 1) // Mainnetへ接続
await zora.totalSupply()
```

### Methods

引数等は[こちら](https://github.com/ourzora/zdk/blob/master/docs/zora.md#read-functions)を参照

#### Read Functions

##### `fetchContentHash(mediaId: BigNumberish)`

メディアの `contentHash` を取得

##### `fetchMetadataHash(mediaId: BigNumberish)`

メディアの `metadataHash` を取得

##### `fetchContentURI(mediaId: BigNumberish)`

メディアの `contentURI` を取得

##### `fetchMetadataURI(mediaId: BigNumberish)`

メディアの `metadataURI` を取得

##### `fetchCreator(mediaId: BigNumberish)`

メディアの `creator` を取得

##### `fetchCurrentBidShares(mediaId: BigNumberish)`

メディアの現在の `bidShares` を取得

##### `fetchCurrentAsk(mediaId: BigNumberish)`

メディアの現在の販売価格 `Ask` を取得

##### `fetchCurrentBidForBidder(mediaId: BigNumberish, bidder: string)`

メディアの現在の入札者(bidder)の入札額(`Bid`)を取得

##### `fetchPermitNonce(address: string)`

アドレスの次の許可されたnonceを取得
permit nonceとは？

##### `fetchMintWithSigNonces(address: string)`

アドレスの次の `mintWithSig nonce` を取得

##### `fetchBalanceOf(address: string)`

Zoraインスタンスのアドレスのbalanceを取得

##### `fetchOwnerOf(mediaId: BigNumberish)`

Zoraインスタンスのメディアのオーナーを取得

##### `fetchMediaOfOwnerByIndex(owner: string, index: BigNumberish)`

Zoraインスタンスの指定のオーナーの指定インデックスのmediaIdを取得

##### `fetchTotalMedia()`

mintされた有効なメディア(non-burned media)の総数を取得

##### `fetchMediaByIndex(index: BigNumberish)`

インデックスからmediaIdを取得

##### `fetchApproved(mediaId: BigNumberish)`

指定のmediaIdの許可されたアカウント（`approved account`） を取得

##### `fetchIsApprovedForAll(owner: string, operator: string)`

指定されたオペレーターが指定された所有者が所有するすべてのメディアが承認されているかを取得

#### Write Functions

##### `mint(mediaData: MediaData, bidShares: BidShares)`

cryptomedia を新規登録(鋳造)する
bidShares: 制作者に永続的に入る入札料（の手数料）の割合。

##### `mintWithSig(creator: BigNumberish, mediaData: MediaData, bidShares: BidShares, sig: EIP712Signature)`

署名付きメッセージ（signed message）を持って cryptomedia を製作者に変わりに新規登録(鋳造)する
sig: ブロックチェーン上で検証される eip-712 準拠の署名

##### `updateContentURI(mediaId: BigNumberish)`

メディアの`contentURI`を更新

##### `updateMetadataURI(mediaId: BigNumberish)`

メディアの`metadataURI`を更新

##### `setAsk(mediaId: BigNumberish, ask: Ask)`

メディアの販売価格（`Ask`）を設定

##### `removeAsk(mediaId: BigNumberish)`

メディアの販売価格（`Ask`）を削除

##### `setBid(mediaId: BigNumberish, bid: Bid)`

メディアへの入札（`Bid`）を設定

##### `removeBid(mediaId: BigNumberish)`

メディアへの入札（`Bid`）を削除

##### `acceptBid(mediaId: BigNumberish, bid: Bid)`

メディアへの入札（`Bid`）を承認

##### `permit(spender: string, mediaId: BigNumberish, sig: EIP712Signature)`

メディアの所有者に代わって指定のアドレスがmediaを使用できるようにする
spender: メディアを使うことが許可されるアドレス

##### `burn(mediaId: BigNumberish)`

mediaを買い戻す?(burn)

##### `revokeApproval(mediaId: BigNumberish)`

承認を取り消す

##### `approve(to:string, mediaId:BigNumberish)`

指定されたメディアの指定されたアドレスへの承認を付与する
to: メディアの承認を得るアドレス

##### `setApprovalForAll(operator: string, approved: boolean)`

msg.senderによって全メディア所有者への承認の付与
operator: `approvalForAll`がセットされるアカウントのアドレス
approved: operatorに対して承認を許可するかしないか?

##### `transferFrom(from: string, to: string, mediaId: BigNumberish)`

指定のアドレスへ指定のメディアを所有権を移す

##### `safeTransferFrom()`

ERC721-Receiver Interface に準拠している場合に限り、 指定のアドレスへ指定のメディアを所有権を移す
