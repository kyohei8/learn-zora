# Docs

## Utils

ユーティリティー関数は大きく分けて3つのカテゴリに分けられます。

- 型コンストラクタ
- ハッシュユーティリティ
- EIP-712 ユーティリティ

### Type Constructors

Zoraインスタンスで利用するいくつかの型を生成するためのメソッド。
型のコンストラクタは入力を受け付け、検証を行い、適切にフォーマットされた Zoraの型を返します。

#### `constructMediaData(tokenURI: string, metadataURI: string, contentHash: BytesLike, metadataHash: BytesLike)`

- `MediaData` 型を生成する
- URIが `https://` で始まることを検証
- ハッシュの長さが正確に32バイトであることを検証

```ts
const contentHash = await sha256FromBuffer(Buffer.from('some content'))
const metadataHash = await sha256FromBuffer(Buffer.from('some metadata'))

const mediaData = constructMediaData(
  'https://token.com',
  'https://metadata.com',
  contentHash,
  metadataHash
)
/*  MediaData =>
{
  metadataURI: "https://ipfs.io/ipfs/QmY..",
  tokenURI: "https://ipfs.io/ipfs/QmVy...",
  contentHash: "0x9e783...",
  metadataHash: "0xe549c...",
}
*/
```

#### `constructAsk(currency: string, amount: BigNumberish)`

通貨アドレスをパースし、有効な Ethereum アドレスであることを確認しaskを生成します。

```ts
const dai = '0x6B175474E89094C44Da98b954EedeAC495271d0F'
const decimal100 = Decimal.new(100)
const ask = constructAsk(dai, decimal100.value)
```

#### `constructBid(currency: string, amount: BigNumberish, bidder: string, recipient: string, sellOnShare: number)`

Bidを生成。 入札者、受取人、通貨のアドレスをパースして検証する。
SellOnShareは4桁に丸められる。

```ts
const dai = '0x6B175474E89094C44Da98b954EedeAC495271d0F'
const decimal100 = Decimal.new(100)
const bidder = '0xf13090cC20613BF9b5F0b3E6E83CCAdB5Cd0FbD5'

const bid = constructBid(dai, decimal100.value, bidder, bidder, 33.3333)
```

#### `constructBidShares(creator: number, owner: number, prevOwner: number)`

bidSharesを生成。
引数に整数を受け取り、Etherの18桁のBigNumberに変換。
BigNumber形式で合計が100になるかを検証。  

```ts
const bidShares = constructBidShares(10, 90, 0)
```

### Hash Utilities

Zoraプロトコルで作成されたすべてのメディアは、コンテンツとメタデータの両方の sha256 ハッシュをブロックチェーンに刻むことになる。
そのため、Zoraと対話する開発者が、あらゆるタイプのサイズのデータのハッシュを作成および検証するための信頼できることが重要になる。

#### `sha256FromBuffer(buffer: Buffer)`

Buffer オブジェクトから sha256 を生成する

```ts
const buf = await fs.readFile('path/to/file')
const hash = sha256FromBuffer(buf)

const otherBuffer = Buffer.from('someContent')
const otherHash = sha256FromBuffer(otherBuffer)
// => 0x764fe5...
```

#### `sha256FromHexString(data: string)`

hex stringから sha256 を生成する。

```ts
const buf = Buffer.from('someContent')
const hexString = '0x'.concat(buf.toString('hex'))
const hash = sha256FromHexString(hexString)
// => 0x764fe5...
```

#### `sha256FromFile(pathToFile: string, chunkSize: number)`

ローカルファイルから sha256 を生成。  
readStreamを利用して、大きなファイルのハッシュ化に有効。

```ts
const hash = await sha256FromFile('path/to/file', 16 * 1024)
```

### EIP-712 Utilities

#### `signPermitMessage(owner: Wallet, toAdderss: string, mediaId: number, nonce: number, deadline: number, domain: EIP712Domain)`

ERC-20標準の拡張として `Permit` が指定され、ユーザが `ETH` を必要とせずに `approval` を発行できるようになった。  
我々はそれをZoraプロトコルで使用するためにさらに拡張し、
ユーザが承認された`apppeved`ユーザがであれば以下のアクションを、他のスマートコントラクトに承認を委任できるようにしました。

- setAsk
- acceptBid
- updateContentURI
- updateMetadataURI
- transfer

今のところ、Signer は ethers Wallet オブジェクトでなければなりません。しかし、近いうちにどんなSignerでもサポートするようになる。

```ts
const provider = new JsonRpcProvider()
const [mainWallet, otherWallet] = generatedWallets(provider)
const rinkebyZora = new Zora(mainWallet, 4)
const deadline = Math.floor(new Date().getTime() / 1000) + 60 * 60 * 24 // 24 hours
const domain = rinkebyZora.eip712Domain()
// => { chainId: 1337, name: "Zora", verifyingContract: "0x6Be6..." (Mediaコントラクトのアドレス)
//   version: "1"
//   }

const eipSig = await signPermitMessage(
  mainWallet,
  otherWallet.address,
  0, // mediaId
  0, // nonce
  deadline,
  domain
)
```

#### `signMintWithSigMessage(owner: Wallet, contentHash: BytesLike, metadataHash: BytesLike, creatorShareBN: BigNumber, nonce: number, deadline: number, domain: EIP712Domain)`

EIP-712を拡張して、クリエイターがETHを必要とせずにMintできるようにする。
ユーザーはmintWithSigメッセージに署名し、信頼されたリレイヤーを使用してトランザクションを中継し、自分に代わってMintを行うことができます。

```ts
const provider = new JsonRpcProvider()
const [mainWallet, otherWallet] = generatedWallets(provider)
const rinkebyZora = new Zora(otherWallet, 4)

const contentHash = await sha256FromBuffer(Buffer.from('some content'))
const metadataHash = await sha256FromBuffer(Buffer.from('some metadata'))
const contentURI = 'https://token.com'
const metadataURI = 'https://metadata.com'

const mediaData = constructMediaData(contentURI, metadataURI, contentHash, metadataHash)
const bidShares = constructBidShares(10, 90, 0)
const deadline = Math.floor(new Date().getTime() / 1000) + 60 * 60 * 24 // 24 hours
const domain = rinkebyZora.eip712Domain()
const nonce = await rinkebyZora.fetchMintWithSigNonce(mainWallet.address)
const eipSig = await signMintWithSigMessage(
  mainWallet,
  contentHash,
  metadataHash,
  Decimal.new(10).value,
  nonce.toNumber(),
  deadline,
  domain
)
```

#### `recoverSignatureFromPermit(owner: Wallet, toAddress: string, mediaId: number, nonce: number, deadline: number, domain: EIP712Domain, sig: EIP712Signature)`

`Permit`メッセージの署名秘密鍵のアドレスを復元する

```ts
const provider = new JsonRpcProvider()
const [mainWallet, otherWallet] = generatedWallets(provider)
const zora = new Zora(provider, 4)
const deadline = Math.floor(new Date().getTime() / 1000) + 60 * 60 * 24 // 24 hours
const domain = zora.eip712Domain()
const eipSig = await signPermitMessage(
  mainWallet,
  otherWallet.address,
  1,
  1,
  deadline,
  domain
)

const recovered = await recoverSignatureFromPermit(
  otherWallet.address,
  1,
  1,
  deadline,
  domain,
  eipSig
)

if (recovered.toLowerCase() != mainWallet.address.toLowerCase()) {
  console.log('Unable to Validate Signature')
} else {
  console.log('Signature Validated')
}
```

#### `recoverSignatureFromMintWithSig(owner: Wallet, contentHash: BytesLike, metadataHash: BytesLike, creatorShareBN: BigNumber, nonce: number, deadline: number, domain: EIP712Domain, sig: EIP712Signature)`

`mintWithSig`メッセージの署名秘密鍵のアドレスを復元する

```ts
const provider = new JsonRpcProvider()
const [mainWallet] = generatedWallets(provider)
const rinkebyZora = new Zora(provider, 4)
const deadline = Math.floor(new Date().getTime() / 1000) + 60 * 60 * 24 // 24 hours
const domain = rinkebyZora.eip712Domain()

const contentHash = await sha256FromBuffer(Buffer.from('some content'))
const metadataHash = await sha256FromBuffer(Buffer.from('some metadata'))

const eipSig = await signMintWithSigMessage(
  mainWallet,
  contentHash,
  metadataHash,
  Decimal.new(10).value,
  1,
  deadline,
  domain
)

const recovered = await recoverSignatureFromMintWithSig(
  contentHash,
  metadataHash,
  Decimal.new(10).value,
  2,
  deadline,
  domain,
  eipSig
)

if (recovered.toLowerCase() != mainWallet.address.toLowerCase()) {
  console.log('Unable to Validate Signature')
} else {
  console.log('Signature Validated')
}
```
