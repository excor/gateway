# Gateway #

Gatewayは、EXCプラットフォームへの単純化された使いやすいインターフェースを提供します。このページでは、EXCプラットフォームで送金をするためのAPIの使用方法について説明します。


## API ##

#### Accounts ####

* [Generate Account - `GET /v1/accounts/new`](#generate-account)
* [Get Account Balance - `GET /v1/accounts/{:account_id}/balance`](#get-account-balance)

#### Payments ####

* [Submit Payment - `POST /v1/accounts/{:account_id}/payments`](#submit-payment)
* [Confirm Payment - `GET /v1/accounts/{:account_id}/payments/{:transaction_id}`](#confirm-payment)
* [Get Payment History - `GET /v1/accounts/{:account_id}/payments`](#get-payment-history)


## EXC Platform Overview ##


### EXC Platform Concepts ###

EXCプラットフォームはデジタルで決済を行うシステムです。EXCプラットフォームは、様々なデジタル通貨をリアルタイムに決済をすることができます。

EXCプラットフォームでは、各オーナーは`Account ID`を振り分けられ、それによって識別されます。`Account ID`はアカウントを一意に識別する文字列です。次に例を示します。`0xDE092383A1B3CC71`

送金は、2つのアカウント間で送金元`Account ID`と送金先`Account ID`を指定することによって行われます。送金には送金する通貨と送金額を指定する必要があります。


### About Excor Account ###

EXCアカウントは事業者用アカウントと利用者用アカウントの二種類があります。

事業者用アカウントはExcorによって作成・管理されます。事業者用アカウントを作成する方法についてはExcorオンラインサポート(support@excor.org)にお問い合わせください。


#### 事業者用アカウントで可能な事 ####
* 事業者の顧客の利用者用アカウントを事業者のサブアカウントとして作成する。
* 事業者用アカウント及び作成した利用者用アカウントの残高を取得する。
* 事業者用アカウント及び作成した利用者用アカウントから送金を行う。
* 事業者用アカウント及び作成した利用者用アカウントの履歴を取得する。

#### 利用者用アカウントで可能な事 ####
* 利用者用アカウントの残高を取得する。
* 利用者用アカウントから送金を行う。
* 利用者用アカウントの履歴を取得する。


### About Gateway ###

Gatewayは、既存のAPIをEXCプラットフォームのプロトコルに変換するプロトコルコンバーターの役割を果たします。現状はREST APIが用意されています。

EXCアカウントのセキュリティ情報は`HSM`に保管され、Gatewayからの要求に応じてトランザクションの電子署名及び電子署名の検証を行います。


### About HSM ###

`HSM`はEXCアカウントのセキュリティ情報の保護に特化して設計された専用モジュールです。 `HSM`は電子署名及び電子署名の検証に使用されます。セキュリティ情報を安全に管理・処理し、耐タンパー性を備えたモジュール内に保管します。

Gatewayを使用するためには`HSM`が必要となります。

`HSM`は事業者用アカウントを作成された事業者にExcorから提供されます。


# Running Gateway #


## Using Gateway ##

Gatewayを使用するには、有効なEXCアカウントが必要です。

開発者は安全なHTTP GETおよびPOST要求を行えるようにするHTTPクライアントも必要です。テストのために以下の選択肢があります。

 * The [`curl`](http://curl.haxx.se/) commandline utility
 * The [Advanced REST client Chrome extension](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=en)



# Formatting Conventions #

現状のGatewayはREST APIの動作に準拠したAPIを提供します。

* APIエンドポイントにHTTP要求を送信し、URL自体の中に必要なリソースを示します。
* HTTPメソッドは実行したい処理を識別します。一般に、HTTP GET要求は情報の取得に使用され、HTTP POST要求は情報の変更または送信に使用されます。
* より複雑な情報を送信する必要がある場合は、HTTP POSTリクエストの本文内にJSON形式のデータとして含められます。
    * これはContent-Type: application/jsonをヘッダに設定しなければならないことを意味します。
* 正常に完了すると、サーバーはHTTPステータスコード`200 OK`を返します。応答の本文は、エンドポイントから返された情報を含むJSON形式のオブジェクトになります。

追加の規則として、Gatewayからのすべての応答に`"success"`という成功したかどうかを示すブール値のフィールドが含まれています。


## Errors ##

エラーが発生すると、サーバーはエラーのタイプに応じて、400〜599の範囲のHTTPステータスコードを返します。応答の本文には、問題の原因に関するより詳細な情報が含まれています。

* 200-299の範囲のコードは成功を示します。
    * 特に明記されていない限り、APIは`200 OK` が返されます。
    
* 400-499の範囲のコードは、リクエストが無効か間違っていたことを示しています。
    * `400 Bad Request`JSON本体が不正な形式の場合に発生します。
    * `404 Not Found`指定されたパスが存在しないか、またはそのメソッドをサポートしていない場合に発生します。
    
* 500-599の範囲のコードは、サーバーに問題が発生したことを示します。これは、ネットワークの停止やどこかのソフトウェアのバグが原因である可能性があります。
    * `500 Internal Server Error`サーバーがエラーをキャッチしない場合に発生します。これは常にバグです。
    * `502 Bad Gateway`GatewayがEXCプラットフォームに接続できない場合に発生します。
    * `504 Gateway Timeout`EXCプラットフォームがGatewayに応答するのに時間がかかりすぎる場合に発生します。
    
可能であれば、サーバーはエラーに関する詳細情報をJSON応答本体に提供します。これらの応答には、次のフィールドが含まれます。

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | `false` でエラーが発生したことを示します。 |
| error_code | String | 発生したエラーの一般的なカテゴリを示す短いコード。 |
| error | String | 人間が判読可能なエラーの要約。 |
| message | String | （省略可）人間が理解しやすいエラーの説明。 |

## Quoted Numbers ##

Gatewayは大きな数値を指定する必要がある場合は、常にJSON数値型の代わりにJSON文字列を使用します。アプリケーションはこれらの数値を適切な精度で数値型に解析する必要があります。

どの程度の精度が必要かが明確でない場合は、任意精度のデータ型を使用することをお勧めします。

## Currency Amounts ##

EXCアカウントでは金額の指定に小数点を使用しません。金額はすべて整数で指定されます。
例:`1EXC`はEXCプラットフォームの中では`"1000000000"`と指定されます。`0.1EXC`はEXCプラットフォームの中では`"1000000000"`と指定されます。

アプリケーションは金額を適切な数で除算して使用する必要があります。アプリケーションは金額がどの程度の精度が必要なのかを的確に判断し、適切なデータ型を使用する必要があります。


# API #

## Generate Account ##

Gatewayに設定された事業者用アカウントに紐づくサブアカウントを生成します。

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/new
```

<!-- </div> -->

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 34890239,
    "timestamp" : 1546196333,
    "account_id": "0xAABBCCDD11223344"
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | アカウントの生成に成功すると`true`が返ります。 |
| transaction_id | Number | アカウントの生成を行ったトランザクションの`Transaction ID`を返します。 |
| timestamp | Number | アカウントの生成を行ったトランザクションの`Timestamp(UNIX TIME)`を返します。 |
| account_id | String | 新たに生成されたアカウントの`Account ID`を返します。 |


## Get Account Balance ##

指定されたアカウントの現在の残高を取得します。

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/balance
```

<!-- </div> -->

このAPIエンドポイントには、以下のURLパラメーターが必要です。

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | 残高を取得するアカウントの`Account ID`。 |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 92348910,
    "timestamp" 1546197573: 
    "balance": "104629877312"
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | 残高の取得に成功すると`true`が返ります。 |
| transaction_id | Number | 残高の取得を行った時の`Transaction ID`を返します。 |
| timestamp | Number | 残高の取得を行った時の`Timestamp(UNIX TIME)`を返します。 |
| balance | String | アカウントの残高を返します。 |


## Submit Payment ##

アカウントから他のアカウントへ送金を行います。

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
POST /v1/accounts/{:account_id}/payments
```

<!-- </div> -->

このAPIエンドポイントには、以下のURLパラメーターが必要です。

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | 送金元となるアカウントのAccount ID。 |

#### Request Body ####

```js
{
    "payment": [
        {
            "destination_account_id": "0x99887766FFEEDDCC",
            "value" : "100001"
        },
        {
            "destination_account_id": "0xFF00EE11DD22CC33",
            "value": "123456"
        }
        ...
    ]
}
```

リクエストボディは、次のフィールドを持つJSONオブジェクトである必要があります。

| Field | Type | Description |
|-------|------|-------------|
| destination_account_id | String | 送金先のアカウントの`Account ID`。 |
| value | String | 送金を行う金額。 |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 3982012,
    "timestamp" 1546198264: 
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | 送金に成功すると`true`が返ります。 |
| transaction_id | Number | 送金を行ったトランザクションの`Transaction ID`を返します。 |
| timestamp | Number | 送金を行ったトランザクションの`Timestamp(UNIX TIME)`を返します。 |


## Confirm Payment ##

送金の履歴を取得します。

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/payments/{:transaction_id}
```

<!-- </div> -->

このAPIエンドポイントには、以下のURLパラメーターが必要です。

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | 送金に関与するアカウントの`Account ID`。 |
| transaction_id | Number | 送金を行ったトランザクションの`Transaction ID`。 |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 82948234,
    "timestamp" : 1546198964,
    "payment": [
        {
            "source_account_id": "0xAABBCCDDEE11223344",
            "destination_account_id": "0x99887766FFEEDDCC",
            "value": "129831"
        },
        {
            "source_account_id": "0xAABBCCDDEE11223344",
            "destination_account_id": "0xFF00EE11DD22CC33",
            "value": "349834"
        }
        ...
    ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | 送金に成功すると`true`が返ります。 |
| transaction_id | Number | 取得した履歴のトランザクションの`Transaction ID`を返します。 |
| timestamp | Number | 取得した履歴のトランザクションの`Timestamp(UNIX TIME)`を返します。 |
| source_account_id | String | 送金元のアカウントの`Account ID`。 |
| destination_account_id | String | 送金先のアカウントの`Account ID`。 |
| value | String | 送金された金額。 |


## Get Payment History ##

指定されたアカウントに影響を与えた送金の履歴を取得します。

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/payments
```

<!-- </div> -->

このAPIエンドポイントには、以下のURLパラメーターが必要です。

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | 送金に関与するアカウントの`Account ID`。 |

オプションとして次のパラメーターをURLクエリパラメータに含めることができます。

| Field | Type | Description |
|-------|------|-------------|
| source_account_id | String | 指定されている場合は、指定されたアカウントからの送金のみを含めます。 |
| destination_account_id | String | 指定されている場合は、特定のアカウントで受け取った送金のみを含めます。 |
| order | String | `"old"`の場合は、`transaction_id`が古い順に履歴をソートします。`"new"`の場合は新しい順に履歴をソートします。デフォルトは"new"です。 |
| start_transaction_id | Number | 指定されている場合は、指定された`transaction_id`より古い履歴を除外します。 |
| end_transaction_id | Number | 指定されている場合は、指定された`transaction_id`より新しい履歴を除外します。 |
| limit | Number | 一度に返される履歴の最大数を指定します。デフォルトは10です。最大で100まで指定できます。 |

#### Response ####

```js
{
    "success": true,
    "payments" : [
        {
            "transaction_id" : 948302234,
            "timestamp" : 1546199715,
            "payment": [
                {
                    "source_account_id": "0xAABBCCDDEE11223344",
                    "destination_account_id": "0x99887766FFEEDDCC",
                    "value": "129384"
                },
                {
                    "source_account_id": "0xAABBCCDDEE11223344",
                    "destination_account_id": "0xFF00EE11DD22CC33",
                    "value": "123912"
                }
                ...
            ]
        }
        ...
    ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | 送金に成功すると`true`が返ります。 |
| transaction_id | Number | 取得した履歴のトランザクションの`Transaction ID`を返します。 |
| timestamp | Number | 取得した履歴のトランザクションの`Timestamp(UNIX TIME)`を返します。 |
| source_account_id | String | 送金元のアカウントの`Account ID`。 |
| destination_account_id | String | 送金先のアカウントの`Account ID`。 |
| value | String | 送金された金額。 |

