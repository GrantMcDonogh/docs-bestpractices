---
summary: アプリケーションでアクセス制御リスト（ACL）を使用して、特定のユーザープロファイルに基づいたデータへの条件付きアクセスを設定する方法について説明しています。 
tags: best-practices; security-model
en_title: Use Access Control List (ACL) to set up permission-based access to data
---

# アクセス制御リスト（ACL）を使用して権限に基づいたデータアクセスを設定する

<pre class="script-css">
.custom-acl-table th, .custom-acl-table td  {
    text-align: center;
}
</pre>

## ACLとは

アクセス制御リスト（ACL）とは、オブジェクトにアタッチされた権限のリストのことです。ACLには、どのオブジェクトがどのユーザーまたはユーザーのグループに対して許可されているかが保存されます。一般的に、特定のユーザープロファイルによる財務データへのアクセス、ユーザーの部署/部門に基づくデータアクセス、階層アクセスを設定するときにACLを使用します。

OutSystemsでは、[OutSystemsのユーザーロール](https://success.outsystems.com/Documentation/11/Developing_an_Application/Secure_the_Application/User_Roles)を使用して、アプリケーションの特定の画面や操作へのエンドユーザーのアクセスを制限または許可することを推奨しています。 
ただし、アプリケーションデータへの階層化された権限制御を設定する場合は、ユーザーロールの使用を避ける必要があります。ビジネス要件に基づいて拡張性の高い動的なデータ分離を保証するには、ACLを作成することが推奨されます。

ACLが必要なコアサービスの場合、各コアサービスモジュールでデータのアクセスルールを設定するという方法が推奨されます。

![コアサービスのアクセス制御リスト](images/access-control-list.png)

コアサービスモジュールとデータ取得メソッド（WebサービスAPIなど）内のACLロジックはできる限り分離する必要があります。ただし開発者は、コアサービスモジュール以外で使用される公開エンティティの場合、それらのテーブルへのアクセスにACL制御を適用する必要があることを覚えておいてください。

このため以下のように、ACLが関連付けられているオブジェクト（エンティティ）をACL追跡リストで追跡することが推奨されます。

<table markdown="1" style="width: 100%;" class="custom-acl-table">
<tr>
<th colspan="3">
ACL追跡リスト
</th>
</tr>
<tr>
<th style="width: 33%">
コアサービスモジュール
</th>
<th style="width: 33%">
エンティティ
</th>
<th style="width: 33%">
ACLエンティティ
</th>
</tr>
<tr>
<td> 
Customer_CS
</td>
<td>
Customer
</td>
<td> 
CustomerACL
</td>
</tr>
<tr>
<td>
Employee_CS 
</td>
<td>
Employee
</td>
<td> 
EmployeeACL
</td>
</tr>
<tr>
<td>
... 
</td>
<td>
...
</td>
<td> 
...
</td>
</tr>
</table>

## ACLパターン

### 新しいACLレコードを作成する

モジュールが適切に分離されている場合、データを作成すると、ACLの設定によって処理が分担されている適切なコアサービスによって所定のルールに基づきデータが処理されます。たとえば、顧客を作成したとき、現在の部署や部門などに基づいて読み取り/書き込みアクセス権を付与することができます。

![新しいACLレコードの作成](images/creating-new-acl-record.png)

### 新しいACLレコードにアクセスする

データへのアクセスでは、ACLに保存されているアクセスルールを考慮する必要があるため、特定のエンティティからデータを取得する場合、適切なACLエンティティへの結合が作成されます。これにより、ACLルールに適合するデータのみが結果セットに含まれます。

![新しいACLレコードへのアクセス](images/accessing-new-acl-records.png)

#### 例

ACLによって制御されるデータのクエリを実行する方法を示します。
以下のクエリでは、ユーザーのチームの売上の階層表示を取得することができます。販売担当者は自分の売上のみを参照することができ、販売担当役員（階層の親）はチーム全体の売上を参照することができます。

![ACLによるSQLの例](images/acl-example.png) 

* ACLは、ユーザーの階層に属するすべてのユーザーのリストを各ユーザーに提供します

* ユーザープロファイルを更新するたびに、ACLリストを再作成します

* 元のAPIを抽象化し、内部構造とコンセプトと適合させます（例: 異なるシステムからの補足情報により顧客のコンセプトを構成するなど）


<div class="info" markdown="1">

この例は、階層化された結果を取得するための実行時アクセラレータを示しています。

</div>

## パフォーマンス

ACLルールのパフォーマンスを向上させるため、アクセス制御の対象を階層の最上位の重要なエンティティのみに限定し、クエリのオーバーヘッドやコードの保守が過剰にならないようにします。 

たとえば、部門レベルでアクセス制御が設定されている場合、部署レベルやユーザーレベルでの設定は不要です。

![ACLの階層の例](images/acl-hierarchical-example.png?width=300)

## ベストプラクティス

ACLが適用されるコアサービスでは、いくつかの規則/ベストプラクティスに従う必要があります。

* ACLテーブルはエンティティと同じ名前にし、後に「ACL」というサフィックスを付けます。

    ![ACLエンティティの命名規則](images/acl-entity.png)

* ACLテーブルの「Public」プロパティはエンティティと同じにする必要があります。可能な場合は、このプロパティの設定を「No」のままにし、コアサービス内でACLルールの適用を完全に制御するようにします。

    ![ACLのPublicプロパティ](images/acl-entity-private.png)

* ACLルールが適用されるエンティティで新しいレコードを作成する場合、ACLレコードも作成する必要があります。アプリケーションを管理しやすくするため、ACLルールの作成ロジックは別のサーバーアクション内に配置する必要があります。

    ![ACL作成サーバーアクション](images/create-acl-logic.png?width=400)

<div class="info" markdown="1">

ACLが適用されるレコードが変更された場合、そのACLルールの再処理が必要になる場合があります。

</div>

* ACLルールがあるエンティティからデータを取得する場合、要求された結果リストを取得するためにエンティティとACLエンティティの結合を作成します。

    ![ACLデータの取得](images/retrieving-acl-data.png?width=400)

* ACLルールを再設定するため、リセットアクションを作成する必要がある場合があります。このアクションには、ACLテーブル全体またはその一部のみを再作成するために必要なロジックを実装します。

    ACLに関連するすべてのアクションを特定のフォルダ内に保存することが推奨されます。

    ![ACLのリセット](images/acl-reset.png?width=500)
