---
date: 2019-10-11
linktitle: About ghost record

title: ［SQL］JavaScriptのオブジェクトを触ってわかったSQLの削除の仕組み
weight: 10
---

## 結論

SQL構文にある`DELETE`直後はデータをゴーストレコードとして設定し、データベース上には存在しているものの無効なレコードになっている。

## そもそもなんでこんなこと調べたの？

初心者向けのJavaScriptの学習教材を作成していました。
DBの代わりにJSオブジェクトを用いて、データのCRUD機能を実装していくというものです。
オブジェクトは下記のように定義しました。

```
var users = [
    {
        id: 1,
        name: 'yamada',
        age: 21,
        job: 'student'
    },
    {
        id: 2,
        name: 'suzuki',
        age: 24,
        job: 'employee'
    }
]
```

CRUDなので当然DELETEも作成したのですが、その時事件は起きました。

```
VM501:1 Uncaught TypeError: Cannot read property 'id' of undefined
```

え...？



該当コードはこちらでした。

```
function deleteUser(id) {

    // バリデーションは省略〜

    // 対象ユーザーを検索
    for (let i = 0; i < users.length; i++) {
        if (users[i].id === id) {
            // 削除
            return delete users[i];
        }
    }

    return '対象のユーザーは見つかりませんでした。';
}
```

対象ユーザーが見つかるまでループして探し、見つかったら削除するものでした。
最初の一回は実行したらうまくいったのですが、２回目からなぜかうまくいかなくなってしまいました。

そして決定的なことに気づいてしまいます。

```
{
        id: 1,
        name: 'yamada',
        age: 21,
        job: 'student'
    },
    empty,
    {
        id: 3,
        name: 'sato',
        age: 28,
        job: 'self-employee'
    }
```

id:2の削除はうまく行き、emptyになっていました。
JSでdeleteメソッドを実行するとemptyになるらしいです（知らなかったな〜）

なのでforループを回した時に`undefined.id`になるのでさっきのようなエラーが出るということでした。

直し方は単純で`undefined`をスキップするようにしました。

```
function deleteUser(id) {

    // バリデーションは省略〜

    // 対象ユーザーを検索
    for (let i = 0; i < users.length; i++) {
        if (users[i] != undefined && users[i].id === id) {
            // 削除
            return delete users[i];
        }
    }

    return '対象のユーザーは見つかりませんでした。';
}
```

そしてここで疑問が生まれます。

「JSオブジェクトは追加の際に`obj.length`すれば自動でIDを付与できるけど、SQLも同じやり方してんのかな〜？だとするとJSオブジェクトのemptyみたいにDBにも削除したレコードの骸はあるはず。。。」

## やっぱりあった。その名も「GHOST_DATA_RECORD」

DELETE直後にはレコードは消えず、ゴーズトレコードとして存在しています。

では、`obj.length`で取得していた次のID付与はどうやって設定しているのか？

## AUTO_INCREMENTの仕組み

`AUTO_INCREMENT`に次にセットされる値がすでに設定されるようになっています。

ちなみに

```
mysql> ALTER TABLE users AUTO_INCREMENT = 10;
```

とやると`10`からセットされるようになります。

おそらく値はレコード作成される時に`+1`ずつされてるのかなと思います。（違っていたらすいません。）

## ゴーストレコードはこのあとどうなる？

バックグラウンドで定期的に稼働しているゴーストクリーンアップタスク（GhostCleanupTask）が実行されたタイミングでデータベース上から完全にデータが削除されます。

ちなみにDBCCコマンドを実行することで手動でゴーストクリーンアップタスクを実行することも可能だそうです。

> ゴーストレコードが存在している場合は、実レコードとして存在した状態になりますので、スロットにもカウントされています。
> ゴーストレコードが削除されると実レコードが消えますので、スロットとしてカウントされなくなります。

## さいごに

SELECTもゴーストレコードはスルーしてるんだろうなということであくまで予測ですが、JSオブジェクトと同じような挙動をSQL上でもしていることがわかりました。

## 参考

[SQL Server の DELETE の基本動作を見てみる](https://blog.engineer-memo.com/2011/04/05/sql-server-%E3%81%AE-delete-%E3%81%AE%E5%9F%BA%E6%9C%AC%E5%8B%95%E4%BD%9C%E3%82%92%E8%A6%8B%E3%81%A6%E3%81%BF%E3%82%8B/)

[MySQLのAUTO_INCREMENTの自動採番の仕組みを考察](https://www.terakoya.work/mysql-auto_increment-setting-howto/)