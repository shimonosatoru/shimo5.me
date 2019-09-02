---
date: 2019-09-02
linktitle: Difference between resource and resources

title: ［Rails］resourceとresourcesの違い
weight: 10
---

## 結論

resourcesは、複数のリソースに対するCRUD処理を行うためのルーティングを生成する。
resourceは、ただ１つのリソースに対するCRUD処理を行うためのルーティングを生成する。

## どんなルーティングになるのか？

railsにはルーティングされているURI一覧を出力する機能があります。

```
$ rake routes
```

それぞれどんなルーティングが出力されるのか見てみます。

### resourcesの場合

```
resources :users
```

```
   Prefix Verb   URI Pattern                             Controller#Action
    users GET    /users(.:format)                        users#index
          POST   /users(.:format)                        users#create
 new_user GET    /users/new(.:format)                    users#new
edit_user GET    /users/:id/edit(.:format)               users#edit
     user GET    /users/:id(.:format)                    users#show
          PATCH  /users/:id(.:format)                    users#update
          PUT    /users/:id(.:format)                    users#update
          DELETE /users/:id(.:format)                    users#destroy
```

リソースが複数ある前提なので`:id`の指定があります。
新規作成や一覧取得には`:id`は必要ないので指定されていません。

### resourceの場合

```
resource :users
```

```
    Prefix Verb   URI Pattern                             Controller#Action
 new_users GET    /users/new(.:format)                    users#new
edit_users GET    /users/edit(.:format)                   users#edit
     users GET    /users(.:format)                        users#show
           PATCH  /users(.:format)                        users#update
           PUT    /users(.:format)                        users#update
           DELETE /users(.:format)                        users#destroy
           POST   /users(.:format)                        users#create
```

resourcesが複数だったのに対し、resourceは一つのリソースに対してなので`:id`の指定がありません。
そもそも一つだから`:id`がなくても編集、削除などができるということですね。

## 参考

[resourceとresourcesの違い](https://qiita.com/ryuuuuuuuuuu/items/e5960c7fecad4ef1301b)
[Railsのresourcesとresourceついて](https://qiita.com/Atsushi_/items/bb22ce67d14ba1abafc5)