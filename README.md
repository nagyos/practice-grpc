
# practice-grpc（Go）

## 行ったこと
1. Protocol Buffers：protoファイルからGoファイルの生成
2. gRPCにおける主な４種類の通信方式（Unary RPC, Server Streaming PRC, Client Streaming RPC, Bidirectional Streaming RPC）の実装
3. interceptor（UnaryServrINterceptor）の実装
     - logging
     - 認証（[go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware)）
     - エラーハンドリング（NotFound, DeadlineExceeded）
4. SSL通信の実装（[mkcert](https://github.com/FiloSottile/mkcert)）

## Protocol Buffers (Google 2008年)
- スキーマ言語：要素や属性などの構造を定義するための言語
- gRPCのデータフォーマットとして使用される
- プログラミング言語に変換可能，またJSONにも変換可能
- バイナリ形式にシリアライズするため，サイズ小→高速通信 ([JSON速度比較記事](https://blog.keepdata.jp/entry/2018/04/06/103104))
- 型があるため，安全性高い

## gRPC (Google 2015)
- データフォーマットにProtocol Buffersを使用
     - 型付けされたデータ転送
     - バイナリにより高速な通信と送信データ量小
- IDL（Protocol Buffers）からserver側・client側に必要なソースコードを生成
- HTTP/2通信を使用
- 特定の言語やプラットフォームに依存しない
### Service
- RPC（メソッド）の実装単位
     - サービス内にメソッドがエンドポイントになる
     - １サービス内に複数のメソッドを定義できる 
- コンパイル後はインターフェースとなる（Go）
     - アプリケーション側でこのインターフェースを実装する 
### gRPCの通信方式
- Unary RPC (1 request 1 reposense)
     - 用途：APIなど 
- Server Streaming PRC (1 request N reposense)
     - 用途：サーバからのプッシュ通知など
     - 返り値側に"stream"を記述する（.proto）
- Client Streaming PRC (N request 1 reposense)
     - 用途：ファイルのアップロードなど
     - 引数側に"stream"を記述する（.proto）
- Bidrectional Streaming PRC (N request N reposense)
     - 用途：チャットやオンライン対戦ゲームなど
     - 返り値と返り値の両方に"stream"を記述する（.proto）

### Interceptor
- メソッドの前後に処理を行うための仕組み
- 認証やロギング，監視やバリデーションなど複数のRPCで共通して行いたい処理で使用する
- Unary用とstreaming用が用意されている
- server側・client側に対応
     - server側
          - UnaryServerInterceptor
          - StreamServerInterceptor
     - client側  
          - UnaryClientInterceptor
          - StreamClientInterceptor
- server側では，```grpc.NewServer```の引数にオプションとして，```grpc.UnaryInterceptor(myInterceptor())```のように追加
- client側では，```grpc.Dial```の引数にオプションとして，```grpc.WithUnaryInterceptor(myInterceptor())```のように追加
