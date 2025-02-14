## Application Service > Cheating Detection > APIガイド
- Cheating Detection APIを使用するために必要なAPIを説明します。
## API共通情報

### 事前準備
- APIを使用するにはプロジェクト統合AppkeyまたはサービスAppkeyが必要です。 
- サービスAppkeyはコンソール上部のURL & Appkeyメニューで確認できます。

### リクエスト共通情報
- APIを使用するにはセキュリティキー認証処理が必要です。

[APIドメイン]

 | 環境 | ドメイン | 
 | --- | --- | 
 | Real | http://ctd-api.cloud.toast.com |

## 認証API

### アクセストークン(token)発行API

- 不正行為検知APIのリクエスト時に必要なアクセストークン発行API

```
URL : /auth/token?appKey={appkey}&expiresIn={expiresIn}
METHOD : POST
Content-type : application/json;charset=utf-8
```

##### リクエスト

[URL Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| expiresIn | Integer | トークン有効時間(秒) | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| userId | String | ユーザーID(受験生番号) | O |
| examNo | String | 試験番号 | O |
| deviceType | String | デバイス区分(pc：PC、mo：Mobile ) | O |
| webAuth | JSON | WebAuth認証データ | O |
| webAuth.userId | String | ユーザーID(受験生番号) | O |
| webAuth.token | String | **<span style="color:#e11d21">顧客が発行したWebAuth認証トークン</span>** | O |
| webAuth.via | String | その他情報 | X |

[リクエスト本文例]

``` json
{
    "userId" : "user123",
    "examNo" : "21342",
    "deviceType" : "pc",
    "webAuth" : {
        "userId": "user123",
        "token": "ADs3Fsdfasdfnv23fkja34FX=",
        "via": ""
    }
}
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 |
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード(0：成功、その他：失敗) |
| header.resultMessage | String | リクエスト結果メッセージ |
| data.tokenType | String | トークンタイプ、 bearerで固定 |
| data.accessToken | String | ユーザーアクセストークン |
| data.expiresIn | Integer | アクセストークンの有効期限(秒) |

[レスポンス本文例]

``` json
{
    "header" : {
        "isSuccessful" : true,
        "resultCode" : 0,
        "resultMessage" : "Success"
    },
    "data" : {
        "tokenType" : "bearer",
        "accessToken" : "DfA3f3da3/34SF+edf6898D2343fsasdf3f=",
        "expireIn" : 600
    }
}
```

### アクセストークンキャンセルAPI

- 発行されたアクセストークンのキャンセル\(強制的に無効にするための\) API

``` yaml
URL : /auth/revoke
METHOD : POST
Content-type : application/json;charset=utf-8
```

##### リクエスト

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| accessToken | String | キャンセルするアクセストークン | O |

[リクエスト本文例]
``` json
{
   "accessToken" : "XDadhaS3dvns34Fdfnf23=="  
}
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード(0：成功、その他：失敗) |
| header.resultMessage | String | リクエスト結果メッセージ |
| data.accessToken | String | キャンセルしたユーザーアクセストークン |

[レスポンス本文例]

``` json
{
    "header" : {
        "isSuccessful" : true,
        "resultCode" : 0,
        "resultMessage" : "Success"
    },
    "data" : {
        "accessToken" : "XDadhaS3dvns34Fdfnf23=="
    }
}
```

## 不正行為検知リクエストAPI

### 行動検知リクエストAPI
- 行動検知を分析するためのリクエストAPI
```
URL : /nhn-behavior-det/v1.0/appkeys/{appKey}/exam/{examNo}/users/{userId}?camLocation={camLocation}&reqTime={reqTime}
METHOD : POST
X-Auth-Token : Bearer {accessToken}
Content-type : multipart/form-data
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-Auth-Token | String | AccessToken | O |

[URL Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| camLocation | String | カメラ位置情報(side(側面)、front(正面)) | O |
| reqTime | long | リクエスト時間(timestamp 10桁)(秒単位まで) | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| file | Binary | 画像ファイル <br>画像推奨事項 <br> side (Size：1280 x 720、拡張子：jpg, jpeg) <br>front (Size：640 x 480、拡張子：jpg, jpeg) | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| examNo | String | 試験番号 | O |
| userId | String | ユーザーID(受験生番号) | O |

[リクエスト本文例]
```
curl -X POST "{doamin}/nhn-behavior-det/v1.0/appkeys/{appKey}/exam/{examNo}/users/{userId}?camLocation={camLocation}&reqTime={reqTime}" 
-H "accept: application/json;charset=UTF-8" 
-H "X-Auth-Token: Bearer {accessToken}" 
-H "Content-Type: multipart/form-data" 
-F "file=@testImage.jpeg;type=image/jpeg"
```


##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```

### 音声検知リクエストAPI
- 音声検知を分析するためのリクエストAPI
```
URL : /nhn-voice-det/v1.0/appkeys/{appKey}/exam/{examNo}/users/{userId}?&reqTime={reqTime}
METHOD : POST
X-Auth-Token : Bearer {accessToken}
Content-type : multipart/form-data
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-Auth-Token | String | AccessToken | O |

[URL Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| reqTime | long | リクエスト時間(timestamp 10桁)(秒単位まで) | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| file | Binary | 音声ファイル<br>(サポート形式 .wav, .wave, .webm)<br>(推奨16bit、16,000 sampling rate、mono channel) | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| examNo | String | 試験番号 | O |
| userId | String | ユーザーID(受験生番号) | O |

[リクエスト本文例]
```
curl -X POST "{domain}/nhn-voice-det/v1.0/appkeys/{appkey}/exam/{examNo}/users/{userId}?reqTime={reqTime}" 
-H "accept: application/json;charset=UTF-8" 
-H "X-Auth-Token: Bearer {accessToken}" 
-H "Content-Type: multipart/form-data" 
-F "file=@fileName.wav;type=audio/wav"
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード(0：成功、それ以外：失敗) |
| header.resultMessage | String | リクエスト結果メッセージ |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```

## NHN Proctor API

### Proctorイベント収集API
- Proctorで発生するイベントを収集するAPI
```
URL : /nhn-cht-prt/v1.0/proctor/event
METHOD : POST
X-CD-Client-Type : Proctor
Content-type : application/json;charset=utf-8
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-CD-Client-Type | String | クライアントタイプ「Proctor」固定 | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| userId | String | ユーザーID(受験生番号) | O |
| examNo | String | 試験番号 | O |
| proctorVersion | String | NHN Proctorアプリバージョン情報 | O |
| eventTime | Long | イベント発生時間 | O |
| deviceID | String | UUID形式のデバイス識別子 - アプリインストール時に発行 | O |
| sessionID | String | UUID形式のセッションID - ブラウザローディング時に発行 | O |
| platform | String | OS情報 | O |
| eventSource | String | イベントソース( 「Proctor」固定) | O |
| event | JSON | イベント | O |
| event.status | String | initialize：ログイン、 begintTest：テスト開始、endTest：テスト終了<br><span style="color:#e11d21">\* Window / Mac | O |
| event.keyboard | String | タスクの切り替えを試行(Attempting switch program.)<br><span style="color:#e11d21">\* Window | X |
| event.mouse | String | テスト領域の外へマウス移動検知(テスト場以外の領域に移動ができない状況ですが例外が発生した場合)<br><span style="color:#e11d21">* Window | X |
| event.detection | String | mouseExit：テスト画面の外へマウスを移動した時(フルスクリーン&ミラーリング状態では転送されない) <br>fullScreenExit：フルスクリーン離脱防止状態でフルスクリーンの解除を試行した時<br>processSwitching：タスクの切り替えブロック状態でタスクの切り替えを試行した時<br><span style="color:#e11d21">\* Mac| X
| event.description | String | イベント付加説明 <br><span style="color:#e11d21">\* Mac
| event.additionalEvent | String | その他イベント情報 | X |

<span style="color:#e11d21">**\* イベントのうち1つは必須** </span>

[リクエスト本文例]

``` json
{
    "appKey" : "your_app_key",
    "userId" : "randy",
    "examNo" : "21342",
    "proctorVersion" : "1.0.0.1",
    "eventTime" : 1619485194941,
    "deviceID" : "9faed1a8-964f-4097-a420-c9d9f38ab693",
    "sessionID" : "1a8aad31-cc10-49bc-848d-a02e05075bbd",
    "platform" : "Windows 10(10.0)",
    "eventSource" : "Proctor",
    "event" : {
        "status" : "initialize"
    }
}
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```

### Proctor指標収集API
- Proctorで受験者のPCにインストールされたプログラム情報を収集するAPI
```
URL : /nhn-cht-prt/v1.0/proctor/collect
METHOD : POST
X-CD-Client-Type : Proctor
Content-type : application/json;charset=utf-8
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-CD-Client-Type | String | クライアントタイプ「Proctor」固定 | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| installApp | JSON | インストールプロセス収集項目 | O |
| installApp.displayName | String | アプリケーション名 | O |
| installApp.displayVersion | String | バージョン情報 | O |
| installApp.publisher | String | 開発会社名 | O |
| extInfo | JSON | 追加情報 | X |

[リクエスト本文例]

``` json
{
    "installApp" : [
        {
            "displayName" : "DropBox",
            "displayVersion" : "3.18.1",
            "publisher" : "Dropbox, Inc."
        },
        {
            "displayName" : "Google Chrome",
            "displayVersion" : "40.9.2623.111",
            "publisher" : "Google, Inc."
        }
     ]
}
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```


## ユーザー情報登録API

### 中視線情報登録API
 - 視線追跡不正行為の検知時にユーザー(受験者)の視線情報を補正するためのAPI

```
URL : /nhn-behavior-reg/v1.0/appkeys/{appKey}/exam/{examNo}/users/{userId}/gaze
METHOD : POST
X-Auth-Token : Bearer {accessToken}
Content-type : application/json;charset=utf-8
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-Auth-Token | String | AccessToken | O |

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| file | Binary | 画像ファイル <br>推奨事項(Size：640 x 480、拡張子：jpg, jpeg) | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| examNo | String | 試験番号 | O |
| userId | String | ユーザーID(受験生番号) | O |ロック

[リクエスト本文例]
```
curl -X POST "{domain}/nhn-behavior-reg/v1.0/appkeys/{appKey}/exam/{examNo}/users/{userId}/gaze" 
-H "accept: application/json;charset=UTF-8" 
-H "X-Auth-Token: Bearer {accessToken}" 
-H "Content-Type: multipart/form-data" 
-F "file=@testImage.jpeg;type=image/jpeg"
```

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "Success"
    }
}
```



## 設定照会API

### 機器制御設定照会

```
URL : /nhn-cht-cfg/v1.0/appkeys/{appKey}/configuration/device
METHOD : GET
X-CD-Client-Type : Proctor
Content-type : application/x-www-form-urlencoded;charset=utf-8
```

#####

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-CD-Client-Type | String | クライアントタイプ「Proctor」固定 | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |
| data.appKey | String | 統合AppkeyまたはサービスAppkey |
| data.regionCode | String | リージョンコード |
| data.blockMonitorYn | String | 追加モニター遮断設定 |
| data.blockSwitchTaskYn | String | タスクの切り替え遮断設定 |
| data.blockScreenYn | String | フルスクリーン離脱防止設定 |
| data.blockProgramYn | String | アプリケーション/プログラム遮断設定 |

[レスポンス本文例]

``` json
{
    "header":{
        "isSuccessful":true,
        "resultCode": 0,
        "resultMessage": "Success"
    },
    "data": {
        "appKey": "bdyfjdff",
        "resionCode" : "KR1"
        "blockMonitorYn": "Y",
        "blockSwitchTaskYn" :"Y",
        "blockScreenYn" : "Y",
        "blockProgramYn": "Y"
    }
}
```


### 顔検出設定照会

```
URL : /nhn-cht-cfg/v1.0/appkeys/{appKey}/configuration/face
METHOD : GET
X-CD-Client-Type : Proctor
Content-type : application/x-www-form-urlencoded;charset=utf-8
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-CD-Client-Type | String | クライアントタイプ「Proctor」固定 | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |
| data.appKey | String | 統合AppkeyまたはサービスAppkey |
| data.regionCode | String | リージョンコード |
| data.faceDetectionYn | String | 顔認識使用設定 |
| data.faceDetectionThreshold | Integer | 検出された顔の数 |
| data.faceTopAngle | Integer | 顔の角度(上) |
| data.faceBottomAngle | Integer | 顔の角度(下) |
| data.faceLeftAngle | Integer | 顔の角度(左) |
| data.faceRightAngle | Integer | 顔の角度(右) |

[レスポンス本文例]

``` json
{
    "header": {
        "isSuccessful": true,
        "resultCode": 0,
        "resultMessage": "SUCCESS"
    },
    "data": {
        "appKey" : "bdyfjdff",
        "regionCode" : "KR1",
        "faceDetectionYn" : "Y",
        "faceDetectionThreshold" : 1,
        "faceTopAngle": 20,
        "faceBottomAngle" : 20,
        "faceLeftAngle" : 20,
        "faceRightAngle" : 20
    }
}
```


### 顧客URL設定照会

```
URL : /nhn-cht-cfg/v1.0/appkeys/{appKey}/configuration/url
METHOD : GET
X-CD-Client-Type : Proctor
Content-type : application/x-www-form-urlencoded;charset=utf-8
```

##### リクエスト

[Header Parameter]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| X-CD-Client-Type | String | クライアントタイプ「Proctor」固定 | O |

[Path Variable]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |

##### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| header.isSuccessful | Boolean | リクエスト成否 |
| header.resultCode | Integer | リクエスト結果コード |
| header.resultMessage | String | リクエスト結果メッセージ |
| data.appKey | String | 統合AppkeyまたはサービスAppkey |
| data.regionCode | String | リージョンコード |
| data.serviceUrl | String | 顧客サービスURL |
| data.webAuthUrl | String | ユーザー認証連動URL |
| data.webhookUrl | String | 不正行為を検知した時の送信URL |

[レスポンス本文例]

``` json
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "Success"
  },
  "data": {
    "appKey": "bdyfjdff",
    "regionCode": "KR1",
    "serviceUrl": "https://hook.nhnent.com/service",
    "webAuthUrl": "https://hook.nhnent.com/redirect",
    "webhookUrl": "https://hook.nhnent.com/webhook"
  }
}
```

## レスポンスコード

### ResultCode

| コード | 区分 | メッセージ | 説明 |
| --- | --- | --- |--- |
| 0 | 成功 | SUCCESS | 正常処理 |
| -20001 | エラー | アクセストークン期限切れ | アクセストークンが有効期限切れ、またはキャンセルリクエストされたトークンでリクエストした時 |  
| -20002 | エラー | 有効ではないアクセストークン | 有効ではないアクセストークンでリクエストした時 | 
| -20003 | エラー | WebAuthUrl通信不可エラー | WebAuthUrlで通信できない(アクセス不可)の場合に発生 | 
| -20004 | エラー | WebAuth認証失敗 | WebAuth認証に失敗した時 |
| -20005 | エラー | 認証されていないユーザーアクセスエラー | X-Auth-Tokenフィールドがないため認証処理不可 | 
| -20006 | エラー | アクセストークンタイプエラー | X-Auth-TokenヘッダBearerがない時に発生 | 
| -40000 | エラー | 制御オプション未設定エラー | リクエストしたAppKeyの設定情報がDBに存在しない場合に発生 |
| -40002 | エラー | 視線登録失敗 | 視線登録APIリクエスト中にエラーが発生した場合 |
| -40003 | エラー | 画像保存設定情報の登録失敗 | OBS認証を受けられなかった場合に発生 |
| -41000 | エラー | 認証されていないAppKey | 存在しないアプリケーションキーまたは有効になっていない場合に発生 |
| -50000 | エラー | 形式が合っていないファイル | サポートしていないファイルを添付した場合 | 
| -50001 | エラー | リクエストファイルがない | ファイルが添付されていない場合 |
| -50002 | エラー | ファイルサイズ超過 | 添付ファイルが1Gを超えている場合に発生 |
| -50005 | エラー | 添付フィールド抜けエラー | 添付ファイルフィールドがForm Dataにない場合
| -50008 | エラー | リクエスト値エラー[{message}] | リクエストパラメータが無効な場合(メッセージにパラメータ情報含む) |
| -50009 | エラー | サポートされていないContent-Type | リクエストContent-Typeがサポートされていない場合 |
| -50010 | エラー | サポートされていないHttpMethod | リクエストMethodがサポートされていない場合 |
| -50011 | エラー | ファイルサイズ0またはファイル識別が不可 | 添付ファイルが正常にアップロードされなかったかエラーがある場合 |
| -99999 | エラー | 内部エラー | 無効なリクエストか、予期せぬサーバーエラーが発生した場合 |

### HttpStatusCode

| コード | 区分 | コード名 | 説明 |
| --- | --- | --- | --- |
| 200 | 正常 | Ok | 正常 |
| 404 | エラー | Not Found | 存在しないURLを呼び出した時に発生 |
| 500 | エラー | Server Error | サーバーがメンテナンス中または障害が発生している場合に発生 | 

## 顧客設定URL API

### WebAuthURL

- ユーザー認証のために顧客のWebAuthUrlに認証リクエスト**(コンソールでWebAuthURL設定必須)**
- ユーザー\(志願者\)の状態を定期的に確認し、本人であることを確認

```
URL : {webAuthUrl}
METHOD : POST
Content-type : application/json;charset=utf-8
```

#### リクエスト

[Request Body]

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| userId | String | ユーザーID(受験生番号) | O |
| token | String | WebAuth認証トークン | O |
| via | String | その他の情報 | X |
| validation | String | 有効性チェック <br> *ユーザーIP、引き込み時間ts(Unix 13 digit Timestamp)値をAES256暗号化した情報 <br> 例) {"ip":"127.0.0.1", "ts": 1621840609833} JSONテキストをAES256暗号化 | X |

[リクエスト本文例]

``` json
{
    "userId": "user123",
    "token": "asdfasdfnv23fkja..",
    "via": "",
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg=="
}
```

#### レスポンス

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| resultCode | Integer | 認証結果コード(0：成功、それ以外：失敗) |
| resultMessage | String | 認証結果メッセージ |

[レスポンス本文例]

成功

``` json
{
    "resultCode": "0",
    "resultMessage": "Success"
}
```

失敗

``` json
{
    "resultCode": "40000",
    "resultMessage": "トークンの有効期限切れ"
}
```

### WebHookURL

- 分析した画像および音声ファイルから不正行為を検出した時、チート情報を伝達**(コンソールでWebhook URL設定必須)**

```
URL : {webhookUrl}
METHOD : POST
Content-type : application/json;charset=utf-8
```

#### リクエスト

[Request Body] GAZE, POSE, BACKGROUND, AUDIO

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| userId | String | ユーザーID(受験生番号) | O |
| examNo | String | 試験番号 | O |
| cheatGroup | String | チートグループ( GAZE、POSE、BACKGROUND,AUDIOのいずれかをレスポンス) | O |
| cheatLevel | Integer | 不正行為レベル(0：Normal、1：Attention\_Low、2：Attention\_Hight、3：Warning) | O |
| eventTime | Long | イベント発生時間(timestamp) | O |
| fileUrl | String | 画像ファイルまたは音声ファイル保存パス | O |
| cheatData | JSON | 不正行為情報 | O |
| cheatData.cheatInfo | JSON | 不正行為判断結果 | O |
| cheatData.cheatInfo.absence | boolean | 不在かどうか<br />- 視線追跡(顔認識を行うかどうか) <br />- 行動検知(人の数) | X |
| cheatData.cheatInfo.thirdPerson | boolean | 第三者を識別するかどうか(視線追跡使用時) | X |
| cheatData.cheatInfo.facePoseYawOut | boolean | 顔上下角度離脱有無(顔検出使用時) | X |
| cheatData.cheatInfo.facePosePitchOut | boolean | 顔左右角度離脱有無(顔検出使用時) | X |
| cheatData.cheatInfo.eyeGazeYawOut | boolean | 視線上下角度離脱有無(視線追跡使用時) | X |
| cheatData.cheatInfo.eyeGazePitchOut | Boolean | 視線左右角度離脱有無(視線追跡使用時) | X |
| cheatData.cheatInfo.eyeGazeScreenOut | boolean | 視線スクリーン離脱有無(事前視線登録完了時) | X |
| cheatData.cheatInfo.unstableBackground | boolean | 背景の変更有無(身体以外のバックグラウンド変化使用時) | X |
| cheatData.cheatInfo.leftHandNotExistence |Boolean | 左手識別有無(行動検知使用時) | X |
| cheatData.cheatInfo.rightHandNotExistence |Boolean | 右手識別を行うかどうか(行動検知使用時) | X |
| cheatData.gaze | JSON | 視線追跡情報 | X |
| cheatData.gaze.numFaces | Integer | 検出された顔の数 | X |
| cheatData.gaze.facePitch | Integer | 顔の上下角度 | X |
| cheatData.gaze.faceYaw | Integer | 顔の左右角度 | X |
| cheatData.gaze.eyePitch | Integer | 視線の上下角度 | X |
| cheatData.gaze.eyeYaw | Integer | 視線の左右角度 | X |
| cheatData.gaze.screenX | float | 視線X軸位置 | X |
| cheatData.gaze.screenY | float | 視線Y軸位置 | X |
| chaetData.bg[] | List | 背景変化情報 | X |
| chaetData.bg[].isChanged | Boolean | 背景変化有無 | X |
| chaetData.bg[].eventTime | Long |発生時間(timstamp 10桁) |X |
| chaetData.bg[].data | JSON | 背景検知詳細情報 |X |
| chaetData.bg[].data.bgChangeDetFlag | Boolean |背景変化有無検知結果 |X |
| chaetData.bg[].data.allocFlag | Boolean | 背景画像スペース割り当て有無(false：背景検知不可) |X |
| cheatData.pose[] | List | 行動検知情報 |X |
| cheatData.pose[].leftHandNotExistence |Boolean | 左手識別有無(行動検知使用時) |X |
| cheatData.pose[].rightHandNotExistence |Boolean | 右手識別有無(行動検知使用時) |X |
| cheatData.pose[].eventTime | Long | イベント発生時間(検知リクエスト時間) |X |
| cheatData.pose[].data | JSON | 行動検知詳細情報 |X |
| cheatData.pose[].data.numPerson |Integer | 検出された人の数 |X |
| cheatData.pose[].data.lHand | JSON | 左手座標情報 |X |
| cheatData.pose[].data.lHand.xmin |Integer |左手領域のバウンディングボックスの上側頂点の座標情報 |X |
| cheatData.pose[].data.lHand.ymin |Integer |左手領域のバウンディングボックスの下側頂点の座標情報 |X |
| cheatData.pose[].data.lHand.xmax |Integer |左手領域のバウンディングボックスの左側頂点の座標情報 |X |
| cheatData.pose[].data.lHand.ymax |Integer |左手領域のバウンディングボックスの右側頂点の座標情報 |X |
| cheatData.pose[].data.lHand.isDetected |Boolean | 左手検出有無 |X |
| cheatData.pose[].data.rHand | JSON | 右手座標情報 |X |
| cheatData.pose[].data.rHand.xmin |Integer |右手領域のバウンディングボックスの上側頂点の座標情報 |X |
| cheatData.pose[].data.rHand.ymin |Integer |右手領域のバウンディングボックスの下側頂点の座標情報 |X |
| cheatData.pose[].data.rHand.xmax |Integer |右手領域のバウンディングボックスの左側頂点の座標情報 |X |
| cheatData.pose[].data.rHand.ymax |Integer |右手領域のバウンディングボックスの右側頂点の座標情報 |X |
| cheatData.pose[].data.rHand.isDetected |Boolean | 右手検出有無 |X |
| cheatData.pose[].data.face | JSON | 顔座標情報 |X |
| cheatData.pose[].data.face.xmin |Integer |顔領域のバウンディングボックス上側の頂点座標情報 |X |
| cheatData.pose[].data.face.ymin |Integer |顔領域のバウンディングボックス下側の頂点座標情報 |X |
| cheatData.pose[].data.face.xmax |Integer |顔領域のバウンディングボックス左側の頂点座標情報 |X |
| cheatData.pose[].data.face.ymax |Integer |顔領域のバウンディングボックス右側の頂点座標情報 |X |
| cheatData.pose[].data.face.isDetected |Boolean | 顔検出有無 |X |
| cheatConfig | JSON | 設定情報 | O |
| cheatConfig.pose.poseEstimationYn | Boolean | 行動検知使用有無 | X |
| cheatConfig.pose.poseEstimationTime | Integer | 左手/右手座標の未識別時間(N秒) | X |
| cheatConfig.gaze.gazeTrackingYn | String | 視線(瞳孔)追跡使用有無 | X |
| cheatConfig.gaze.gazeTopAngle | Integer | 瞳孔角度(上) | X |
| cheatConfig.gaze.gazeBottomAngle | Integer | 瞳孔角度(下) | X |
| cheatConfig.gaze.gazeLeftAngle | Integer | 瞳孔角度(左) | X |
| cheatConfig.gaze.gazeRightAngle | Integer | 瞳孔角度(右) | X |
| cheatConfig.gaze.regUserGaze | JSON | 登録された視線情報 | X |
| cheatConfig.gaze.regUserGaze.numFaces | Integer | 登録された検出された顔の数 | X |
| cheatConfig.gaze.regUserGaze.facePitch | Integer | 登録された顔の上下角度 | X |
| cheatConfig.gaze.regUserGaze.faceYaw | Integer | 登録された顔の左右角度 | X |
| cheatConfig.gaze.regUserGaze.eyePitch | Integer | 登録された視線上下角度 | X |
| cheatConfig.gaze.regUserGaze.eyeYaw | Integer | 登録された視線の左右角度 | X |
| cheatConfig.gaze.regUserGaze.screenX | Integer | 登録された視線のX座標 | X |
| cheatConfig.gaze.regUserGaze.screenY | Integer | 登録された視線のY座標 | X |
| cheatConfig.face.faceDetectionYn | String | 顔認識使用有無 | X |
| cheatConfig.face.faceDetectionThreshold | Integer | 検出顔数の基準値 | X |
| cheatConfig.face.faceTopAngle | Integer | 顔の角度(上) | X |
| cheatConfig.face.faceBottomAngle | Integer | 顔の角度(下) | X |
| cheatConfig.face.faceLeftAngle | Integer | 顔の角度(左) | X |
| cheatConfig.face.faceRightAngle | Integer | 顔の角度(右) | X |
| cheatConfig.face.regUserGaze | JSON | 登録された視線情報 | X |
| cheatConfig.face.regUserGaze.numFaces | Integer | 登録された検出された顔の数 | X |
| cheatConfig.face.regUserGaze.facePitch | Integer | 登録された顔の上下角度 | X |
| cheatConfig.face.regUserGaze.faceYaw | Integer | 登録された顔の左右角度 | X |
| cheatConfig.face.regUserGaze.eyePitch | Integer | 登録された視線の上下角度 | X |
| cheatConfig.face.regUserGaze.eyeYaw | Integer | 登録された視線の左右角度 | X |
| cheatConfig.face.regUserGaze.screenX | Integer | 登録された視線のX座標 | X |
| cheatConfig.face.regUserGaze.screenY | Integer | 登録された視線のY座標 | X |
| cheatConfig.bg.bgDetectionYn | String | 身体以外のバックグラウンド変化使用有無 | X |
| cheatConfig.bg.bgDetectionTime | Integer | バックグラウンド変化検知時間(N秒) | X |
| validation | String | 有効性チェック <br> *ユーザーIP、引き込み時間ts(Unix 13 digit Timestamp)値をAES256暗号化した情報 <br> 例) {"ip":"127.0.0.1", "ts": 1621840609833} JSONテキストをAES256暗号化 | X |

[リクエスト本文例] POSE, GAZE

``` json
{
    "appKey":"AQJ33tPUaI9Y4lc2IrjX",
    "userId":"usertTest",
    "examNo":"12345",
    "cheatGroup":"POSE",
    "cheatLevel":0,
    "eventTime": 1621828945,
    "fileUrl":"https://alp-api-storage.cloud.toast.com/v1/AUTH_3434343434534234234/cheatingdetection/IMAGE/demoWebTest/test/front/20210707123901682.jpg",
    "cheatData":{
        "cheatInfo":{
            "absence":false,
            "thirdPerson":false,
            "unstableBackground":false,
            "leftHandNotExistence":false,
            "rightHandNotExistence":false
        }
    },
    "cheatConfig":{
        "bg":{
            "bgDetectionYn":"Y",
            "bgDetectionTime":20
        },
        "pose":{
            "poseEstimationYn":"Y",
            "poseEstimationTime":20
        }
    },
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg=="
}
```

``` json
{
    "appKey":"AQJ33tPUaI9Y4lc2IrjX",
    "userId":"userTestId",
    "examNo":"12345",
    "cheatGroup":"GAZE",
    "cheatLevel":3,
    "eventTime": 1621828940,
    "fileUrl":"https://alp-api-storage.cloud.toast.com/v1/AUTH_3434343434534234234/cheatingdetection/IMAGE/demoWebTest/diablo3/test/20210707123901682.jpg",
    "cheatData":{
        "cheatInfo":{
            "absence":true,
            "thirdPerson":false,
            "facePoseYawOut":false,
            "facePosePitchOut":false,
            "eyeGazeYawOut":false,
            "eyeGazePitchOut":false,
        },
        "gaze":{
        "numFaces":0,
        "facePitch":0,
        "faceYaw":0,
        "eyePitch":0,
        "eyeYaw":0,
        "screenX":0.0,
        "screenY":0.0
        }
    },
    "cheatConfig":{
        "gaze":{
            "gazeTrackingYn":"Y",
            "gazeTopAngle":24,
            "gazeBottomAngle":24,
            "gazeLeftAngle":24,
            "gazeRightAngle":24,
            "regUserGaze": {
                "numFaces": 0,
                "facePitch": 0,
                "faceYaw": 0,
                "faceDistance": 0,
                "eyePitch": 0,
                "eyeYaw": 0,
                "screenX": 0,
                "screenY": 0
            }
        },
        "face":{
            "faceDetectionYn":"Y",
            "faceDetectionThreshold":1,
            "faceTopAngle":25,
            "faceBottomAngle":25,
            "faceLeftAngle":25,
            "faceRightAngle":25,
            "regUserGaze": {
                "numFaces": 0,
                "facePitch": 0,
                "faceYaw": 0,
                "faceDistance": 0,
                "eyePitch": 0,
                "eyeYaw": 0,
                "screenX": 0,
                "screenY": 0
            }
        }
    },
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg=="
}
```

[Request Body] AUDIO

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| userId | String | ユーザーID(受験生番号) | O |
| examNo | String | 試験番号 | O |
| cheatGroup | String | チート区分(AUDIO固定) | O |
| eventTime | Long | イベント発生時間(timestamp) | O |
| fileUrl | String | 画像ファイルまたは音声ファイル保存パス | O |
| cheatLevel | Integer | 音声検出を行うかどうか(0：未検知、1：検知) | O |
| cheatData | JSON | 検出情報 | X |
| cheatData.voice | Long[] | 音声検出時間(秒)<br>例) [3,4] > 3、4秒に音声検出 | X |
| validation | String | 有効性チェック <br> *ユーザーIP、引き込み時間ts(Unix 13 digit Timestamp)値をAES256暗号化した情報 <br> 例) {"ip":"127.0.0.1", "ts": 1621840609833} JSONテキストをAES256暗号化 | X |

[リクエスト本文例]

音声検知
``` json
{
    "appKey": "AQJ33tPUaI9Y4lc2IrjX",
    "userId": "usertTest",
    "examNo": "12345",
    "cheatGroup": "AUDIO",
    "senderTime": 1621828948,
    "fileUrl": "https://alp-api-storage.cloud.toast.com/v1/AUTH_3434343434534234234/cheatingdetection/AUDIO/demoWebTest/test/20210707123916831.webm",
    "cheatLevel" : 1,
    "cheatData" : {
        "voice": [1,2,3]
    },
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg==",
}
```

音声未検出

``` json
{
    "appKey":"AQJ33tPUaI9Y4lc2IrjX",
    "userId": "usertTest",
    "examNo":"12345",
    "cheatGroup":"AUDIO",
    "senderTime": 1621828948,
    "fileUrl":"https://toast.cloud.com/20210518193737738.wav",
    "cheatLevel" : 0,
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg=="
}
```

[Request Body] PROCTOR

| 名前 | タイプ | 説明 | 必須かどうか |
| --- | --- | --- | --- |
| appKey | String | 統合AppkeyまたはサービスAppkey | O |
| userId | String | ユーザーID(受験生番号) | O |
| examNo | String | 試験番号 | O |
| cheatGroup | String | チート区分(PROCTOR固定) | O |
| platformOs | String | OS情報 | O |
| eventTime | Long | イベント発生時間(timestamp) | O |
| cheatLevel | Integer | チート発生(1)固定 | O |
| cheatData | JSON | 不正行為情報 | O |
| cheatData.keyboard | String | キーボードイベント | X |
| cheatData.mouse | String | マウスイベント | X |
| cheatData.additional | String | その他のイベント | X |
| cheatConfig | JSON | 設定情報 | O |
| cheatConfig.blockMonitorYn | String | 追加モニターの遮断を行うかどうか | O |
| cheatConfig.blockSwitchTaskYn | String | タスクの切り替えを遮断するかどうか | O |
| cheatConfig.blockScreenYn | String | フルスクリーン離脱を防止するかどうか | O |
| cheatConfig.blockProgramYn | String | アプリケーション/プログラムを遮断するかどうか | O |
| validation | String | 有効性チェック <br> *ユーザーIP、引き込み時間ts(Unix 13 digit Timestamp)値をAES256暗号化した情報 <br> 例) {"ip":"127.0.0.1", "ts": 1621840609833} JSONテキストをAES256暗号化 | X |

[リクエスト本文例]

``` json
{
    "appKey": "AQJ33tPUaI9Y4lc2IrjX",
    "userId": "usertTest",
    "platformOs" :"Windows10",
    "examNo":"12345",
    "cheatGroup": "PROCTOR",
    "eventTime": 1621828940,
    "cheatLevel": 1,
    "cheatData": {
        "keyboard": "Attempting switch program."
    },
    "cheatConfig": {
        "appKey" : "bdyfjdff",
        "blockMonitorYn": "Y",
        "blockSwitchTaskYn":"Y",
        "blockScreenYn":"Y",
        "blockProgramYn":"Y"
    },
    "validation" : "LrXE8YJolAdgNiAKikontAb8aj8YkFf3vl+3oM6hdMVDE5bcmbzNgA9aV4y/ZDLdDpTsEsNtKqzcCxnYZMy2lg=="
}
```

[Response Body]

| 名前 | タイプ | 説明 | 
| --- | --- | --- |
| resultCode | Integer | 認証結果コード(0：成功、それ以外：失敗) |
| resultMessage | String | 認証結果メッセージ |

[レスポンス本文例]

``` json
{
    "resultCode": "0",
    "resultMessage": "Success"
}
```
