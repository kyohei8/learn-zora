# Docs

## Metadata

Zoraプロトコルは、スマートコントラクトでMintされたメディアが、メタデータを指すURIを含んでいることが必須である。

Zoraプロトコルは、そのメタデータの構造については干渉しない。つまり、明示的にブロックチェーンレベルで強制力を持たない。
そのため、media-metadata-schemasリポジトリは、JSON Schemaによって記述されたコミュニティがサポートするメタデータスキーマの正当なソースとして機能し、Zora Development Kit (ZDK)を通して提供されるTypes、Parsers、Generators、Validatorを生成します。

### Generate

#### `generateMetadata(version: stirng, metadata: JSONLikeObject)`

スキーマバージョンといくつかのフォーマットされていない json から json を生成。その際に json はバリデートされ、minifyされてアルファベット順になる。

```ts
import { generateMetadata } from '@zoralabs/zdk'

const metadata = {
  version: 'zora-20210101',
  name: randomName,
  description: randomDescription,
  mimeType: mimeType,
}

const minified = generateMetadata(metadata.version, metadata)
```

### Validate

スキーマバージョンと json を検証し、bool値を返す。

#### `validateMetadata(version: string, metadata: JSONLikeObject)`

```ts
import { validateMetadata } from '@zoralabs/zdk'

const metadata = {
  version: 'zora-20210101',
  name: randomName,
  description: randomDescription,
  mimeType: mimeType,
}

const bool = validateMetadata(metadata.version, metadata)
```

### Parse

スキーマバージョンと文字列を比較し、json を返す。

#### `parseMetadata(version: string, json: string)`

```ts
import { parseMetadata } from '@zoralabs/zdk'

const metadata = `
  {
    version: 'zora-20210101',
    name: randomName,
    description: randomDescription,
    mimeType: mimeType,
  }
`
const parsed = parseMetadata('zora-20210101', metadata)
```
