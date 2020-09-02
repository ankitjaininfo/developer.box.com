---
related_endpoints:
  - post_metadata_queries_execute_read
category_id: metadata
subcategory_id: metadata/5-queries
is_index: false
id: metadata/queries/syntax
rank: 2
type: guide
total_steps: 7
sibling_id: metadata/queries
parent_id: metadata/queries
next_page_id: metadata/queries/pagination
previous_page_id: metadata/queries/create
source_url: >-
  https://github.com/box/developer.box.com/blob/default/content/guides/metadata/5-queries/2-syntax.md
---
# クエリ構文

メタデータクエリAPIのクエリ構文はSQLデータベースのクエリ構文と似ています。契約金額が100ドルを超える契約メタデータテンプレートに一致するすべてのファイルとフォルダに対してクエリを実行するには、以下のメタデータクエリを作成します。

```json
{
  "from": "enterprise_123456.contractTemplate",
  "query": "amount >= :value",
  "query_params": {
    "value": 100
  },
  "fields": [
    "name",
    "metadata.enterprise_123456.contractTemplate.amount"
  ],
  "ancestor_folder_id": "5555"
}
```

この場合、`from`値はメタデータテンプレートの`scope`と`templateKey`を表し、`ancestor_folder_id`はサブフォルダを含む検索範囲となるフォルダIDを表します。

## `fields`パラメータ

デフォルトでは、このAPIで返されるのは、`id`、`type`、および`etag`の値を含む、ファイルまたはフォルダの基本レプリゼンテーションのみです。その他のデータをリクエストするには、`fields`パラメータを使用すると、追加のフィールドや、その項目に関連付けられたメタデータに対してクエリを実行できます。

例:

* `created_by`では、項目を作成したユーザーの詳細が応答に追加されます。
* `metadata.<scope>.<templateKey>`では、`scope`と`templateKey`によって識別されたメタデータインスタンスの基本レプリゼンテーションが返されます。
* `metadata.<scope>.<templateKey>.<field>`では、`scope`と`templateKey`によって識別されたメタデータインスタンスの基本レプリゼンテーションのすべてのフィールドに加え、`field`の名前によって指定されたフィールドが返されます。同じ`scope`および`templateKey`の複数のフィールドを定義できます。

## `query`パラメータ

`query`パラメータは、選択したメタデータインスタンスに対して実行する、SQLに似たクエリを表します。このパラメータは省略可能で、このパラメータを指定しない場合、APIはこのテンプレートに対してすべてのファイルとフォルダを返します。

左側の各フィールド名(`amount`など)は、関連付けられたメタデータテンプレートのフィールドの`key`に一致する必要があります。つまり、関連付けられたメタデータインスタンスに実際に存在するフィールドだけを検索できます。その他のフィールド名を指定するとエラーが発生し、エラーが返されます。

### `query_params`パラメータ

クエリ文字列への動的な値の埋め込みをわかりやすくするために、`:value`のように、コロン構文を使用して引数を定義できます。たとえば、次のように指定された各引数では、`query_params`オブジェクトにそのキーを使用した後続の値が必要です。

```json
{
  ...,
  "query": "amount >= :amount AND country = :country",
  "query_params": {
    "amount": 100,
    "country": "United States"
  },
  ...
}
```

### 論理演算子

クエリでは、以下の論理演算子がサポートされます。

<!-- markdownlint-disable line-length -->

| 演算子         |                                                                                                                                             |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `AND`       | `AND`で区切られたすべての条件が`TRUE`の場合に一致となります。                                                                                                        |
| `OR`        | `OR`で区切られた条件のいずれかが`TRUE`の場合に一致となります。                                                                                                        |
| `NOT`       | 先行する条件が`TRUE`**でない**場合に一致となります。                                                                                                             |
| `LIKE`      | テンプレートフィールドの値がパターンと一致する場合に一致となります。文字列値のみに対応します。詳細については、[パターン一致](#pattern-matching)を参照してください。その他の制限については以下を参照してください。                         |
| `NOT LIKE`  | テンプレートフィールドの値がパターンと一致**しない**場合に一致となります。文字列値のみに対応します。詳細については、[パターン一致](#pattern-matching)を参照してください。その他の制限については以下を参照してください。                    |
| `ILIKE`     | `LIKE`と同じですが、大文字と小文字が区別されません。その他の制限については以下を参照してください。                                                                                        |
| `NOT ILIKE` | `NOT LIKE`と同じですが、大文字と小文字が区別されません。その他の制限については以下を参照してください。                                                                                    |
| `IN`        | テンプレートフィールドの値は、指定された引数のリストのいずれかと等しい場合に一致となります。この形式では、`amount NOT IN (:arg1, :arg2, :arg3)`のように、リスト内の各項目は`query_params`引数として明示的に定義する必要があります。 |
| `NOT IN`    | `IN`に似ていますが、テンプレートフィールドの値は、リストに指定されたどの引数にも一致しません。                                                                                           |
| `IS NULL`   | テンプレートフィールドの値が`null`の場合に一致となります。                                                                                                            |
| `IS NOT`    | テンプレートフィールドの値が`null`でない場合に一致となります。                                                                                                          |

<!-- markdownlint-enable line-length -->

<Message notice>

`ILIKE`演算子を使用した場合を除き、`string`または`enum`フィールドでの一致は、どれも大文字小文字が区別されます。

</Message>

<Message warning>

`LIKE`、`ILIKE`、`NOT LIKE`、および`NOT ILIKE`演算子は、メタデータインスタンスの数が10,000項目を超えるテンプレートでは使用できません。このサイズのクエリには[インデックス](g://metadata/queries/indexes)が必要です。また、これらの演算子はインデックスと互換性がありません。

</Message>

### 比較演算子

クエリでは、以下の比較演算子がサポートされます。

<!-- markdownlint-disable line-length -->

| 演算子  |                                        |
| ---- | -------------------------------------- |
| `=`  | テンプレートフィールドの値が、指定した値と**等しい**ことを表します。   |
| `>`  | テンプレートフィールドの値が、指定した値よりも**大きい**ことを表します。 |
| `<`  | テンプレートフィールドの値が、指定した値よりも**小さい**ことを表します。 |
| `>=` | テンプレートフィールドの値が、指定した値**以上**であることを表します。  |
| `<=` | テンプレートフィールドの値が、指定した値**以下**であることを表します。  |
| `<>` | テンプレートフィールドの値が、指定した値と**等しくない**ことを表します。 |

<!-- markdownlint-enable line-length -->

<Message warning>

ビット単位演算子および算術演算子は、メタデータクエリAPIではサポートされていません。

</Message>

### パターン一致

`LIKE`、`NOT LIKE`、`ILIKE`および`NOT ILIKE`演算子は、パターンに対して文字列が一致するかどうかを照合します。このパターンでは、以下の予約文字をサポートします。

* `%` パーセント記号は0個、1個または複数個の文字を表します。たとえば、`%Contract`の場合、`Contract`、`Sales Contract`は一致しますが、`Contract (Sales)`は一致しません。
* `_` アンダースコアは1文字を表します。たとえば、`Bo_`の場合、`Box`、`Bot`は一致しますが、`Bots`は一致しません。

上記の文字はどちらも、他の文字の前後または文字の間に使用できます。パターンには、複数の予約文字を含めることができます。たとえば、`Box% (____)`の場合は`Box Contract (2020)`が一致します。

クエリの例は次のようになります。`%`でラップされた文字列は`query`属性ではなく`query_params`のリストに含まれていることに注意してください。

```json
{
  ...,
  "query": "country ILIKE :country",
  "query_params": {
    "country": "%United%"
  },
  ...
}
```

<Message notice>

`%`または`_`文字を、その文字として照合する必要がある場合は、エスケープするためにバックスラッシュ文字`\`を使用できます。たとえば、`20\%`の場合は、リテラル値`20%`が一致します。

</Message>