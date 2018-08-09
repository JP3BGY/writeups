# Odinハードウェアパート
配布されたバイナリはnRF52シリーズのファームウェアということで仕様書を取り寄せて読む。

http://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.nrf52%2Fdita%2Fnrf52%2Fchips%2Fnrf52832_ps.html

http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0553a/BABIFJFG.html

Cortex-M4はthumb命令しか実行できないと仕様書に書いてあった(はず)なのでとりあえずfirmwareをobjdumpでディスアセンブルしてファイルに落とし込む。(そこに至るまでにthumb命令のpcの最下位1ビットはただのフラグでメモリアクセスには使われない事に気が付かず時間を結構溶かしてしまった)

はじめは起動から順を追って読む必要があると思っていたのでMBRのVector Tableから順に眺めていたのだが、どう考えてもそれをこなせる量じゃないことに気がついたので路線変更、objdump+grep+radare2でコマンドを処理している部分を探す。

pcapファイルを解析したらBLEを通してコマンドのバイトデータをやり取りしているだけなのがわかった。なので、おそらくswitchかif文で条件分岐している部分があると踏んでディスアセンブルしたファイルに0x91でgrep、radare2で出てきた部分(オフセット0x311e8)のCFGを見たら案の定if文でのコマンドの条件分岐部分を見つける。

Xrefを使って読む場所の目星をつけて他のチームメイトにコード解析を投げる。

(その間Something Revengeに手を付けようとしたが方針がわからず無駄に時間を溶かす)

コードの解析結果、アドレス0x100000a4の値4Byteが隠しコマンドに使われていることとそれ以外のコマンド部分がわかったので、再び仕様書とにらめっこ。このアドレスはDevice Addressが保存されているレジスタ(Flash？)だとわかった。この部分はBLEのアドレスとして使われているのでpcapファイルのlocalhost()が送信しているアドレスの下位4Byteをリトルエンディアンの順番でコマンドに組み込めば答えになる。
```
CBCTF{20afa8c15fe5bc03b701}
```