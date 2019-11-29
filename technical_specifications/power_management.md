---
layout: default
title: パワーマネジメント
parent: 技術情報
nav_order: 5
---
# パワーマネジメント

パワーマネジメント機能を使用することで、消費電力を大幅に削減することができます。
Deguのパワーマネジメントでは、以下の2つを設定することが可能です。

* suspend
* power_down

## 比較
| 状態             | 消費電流[^1]  | 説明                                         |
|:-------------------:|------:|:----------------------------------------------:|
| suspend    | 5uA | RAMのデータを保持します |
| power_down | 1uA | RAMのデータを保持しません |
[^1]:ボタン電池での3.0V供給時

### サンプルスクリプト

Githubの[open-degu/degu-micropython-samples](https://github.com/open-degu/degu-micropython-samples/tree/master/basic/power_management)に、パワーマネジメントのサンプルスクリプトがあります。

* `main_A.py`
  ```
  import machine
  import time
  import degu

  if __name__ == '__main__':

      while True:
          print("Hello! I'm Alice.")
          if (degu.check_update()):
              print("New script is comming! Restarting...")
              machine.reset()

          time.sleep(1)
  ```

* `main_B.py`
  ```
  import machine
  import time
  import degu

  if __name__ == '__main__':
      while True:
          print("Hello! I'm Bob.")
          if (degu.check_update()):
              print("New script is comming! Restarting...")
              machine.reset()

          time.sleep(1)
  ```

まず、DeguをPCにUSBケーブルで接続し、`main_A.py`を`main.py`としてDeguにコピーします。

Deguを再起動すると`main.py`が実行され、1秒間隔で`Hello! I'm Alice.`と表示しながらアップデートを確認します。

```
Hello! I'm Alice.
Hello! I'm Alice.
Hello! I'm Alice.
...
```

この状態で、AWS IoT Core上の対応するDevice Shadowを編集し、`main_B.py`のURLとMD5ハッシュ値を追加します。

```
{
    "desired": {
        "script_user": "https://raw.githubusercontent.com/open-degu/degu-micropython-samples/master/basic/remote_update/main_B.py",
        "script_user_ver": "4f3e01b7dc12348ebbd1acfe11ef6328"
    }
}
```

すると、`degu.check_update()`が1を返し、`machine.reset()`で再起動します。

```
New script is comming! Restarting...
```

再起動後、Deguは`main_B.py`を新たな`main.py`としてダウンロードします。ダウンロードが完了したら、再度再起動し、アップデートされた`main.py`を実行します。

```
Hello! I'm Bob.
Hello! I'm Bob.
Hello! I'm Bob.
...
```
