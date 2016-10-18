#第二回 課題2-1: Cbenchのボトルネック調査

##提出者
氏名：木藤嵩人

##課題内容
Ruby のプロファイラで Cbench のボトルネックを解析しよう。

##解答
まず、cbench-t-kitoディレクトリにおいて端末に以下のコマンドを入力してtremaを起動した。
```
ruby -r profile ./bin/trema run ./lib/cbench.rb &> profile.txt
```

この状態で、他の端末から以下のコマンドを実行して計測を行った。
```
./bin/cbench --port 6653 --switches 1 --loops 10 --ms-per-test 10000 --delay 1000 --throughput
```

これによって得られたprofile情報の一部を以下に示す。
```
  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 94.47   101.71    101.71        1 101710.00 101710.00  Thread#join
 94.47   203.42    101.71      128   794.61   794.61  Kernel#sleep
 94.47   305.13    101.71        2 50855.00 50855.00  TCPServer#accept
 94.47   406.84    101.71        1 101710.00 101710.00  IO.select
  3.12   410.20      3.36   110575     0.03     0.15  BinData::Struct#instantiate_obj_at
  2.51   412.90      2.70    68746     0.04     0.17  BinData::Struct#find_obj_for_name
  2.47   415.56      2.66    97108     0.03     0.03  Kernel#define_singleton_method
```

これより、Thread#join, Kernel#sleep, TCPServer#accept, IO.selectがボトルネックになっていることがわかった。

Thread#joinはThreadのインスタンスメソッドで、join -> selfはスレッド self の実行が終了するまで、カレントスレッドを停止する。一度だけ呼ばれているので、このプログラムはシングルスレッドの処理であると考えられる。

Kernel#sleepはKernelのモジュール関数で、sleep(sec) -> Integerはsec 秒だけプログラムの実行を停止する。

TCPServer#acceptはのTCPServerインスタンスメソッドで、accept -> TCPSocketはクライアントからの接続要求を受け付け、接続した TCPSocket のインスタンスを返す。

IO.selectは与えられた入力/出力/例外待ちの IO オブジェクトの中から準備ができたものを それぞれ配列にして、配列の配列として返す。

