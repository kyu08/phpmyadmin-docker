# 定数の書き方
- 文字列と日付の定数は`'`で囲む
- 数値の定数は囲まない

# Database や Table, カラムなどの名前に使える文字
- 半角アルファベット
- 半角数字
- _

また、名前の最初には必ず半角アルファベットを使わなければならない

# データ型
- CHAR型
  - 固定長文字列
  - 文字数が最大長になるまで半角スペースで埋める

# 空行
以下のように空行を挟むとうまく動かないことがある(らしい)。(けど手元だと普通に動いた)
```sql
select *

from hoge
```

# 重複を省いて表示する
```sql
select distinct team_id from member;
```

# 比較演算子
NULL を含んだ計算の結果はすべて NULL になるので注意
```sql
select id from member where team_id = null
```

とかやっても何もとれない
null 判定したい場合は `is null` とか `is not null` とかをつかう

# null を含む場合の真理値
null を含む真理値は `UNKNOWN` になる。
通常の論理演算が2値論理と呼ばれるのに対し、SQLでの論理演算は3値演算と呼ばれる

# memo
select句と where句で式が重複しちゃうの嫌だけど仕方ないっぽい、、(?)
```sql
select shohin_mei, hanbai_tanka * 0.9 - shiire_tanka as 'rieki'
from Shohin
where (shohin_bunrui = '事務用品' OR shohin_bunrui = 'キッチン用品')
  and (hanbai_tanka * 0.9 - shiire_tanka > 100);
```
# 集約関数
`COUNT(*)` 以外のすべての集約関数は、計算前に NULL を除外している
(なので AVG の分母はもちろん NULL を除外したもの)

# HAVING
GROUP BY で集約したデータに対して条件を指定する

# ビュー
- ビューの実態は SELECT文
- ビューには「ORDER BY句は使えない」「ビューに対する更新は不可能ではないが制限がある」という2つの制限がある

- ビューを用いてビューをつくる「多段ビュー」は避けるべき。
  - なぜならビューをFROM句に指定した時の検索は、
    ①最初にビューに定義された SELECT文 が実行され
    ②その結果に対してビューを FROM句 に指定した SELECT文 が実行される
  ためパフォーマンスが悪いから

# サブクエリ
- 使い捨てのビューのようなもの。ビューとは異なり、 SELECT文 の実行終了後に消去される
- サブクエリには名前をつける必要がある
- スカラ・サブクエリとは、必ず1行1列だけの結果を返すという制限をつけたサブクエリのこと

- 以下のように FROM句 の中に SELECT文 を書く
```sql
SELECT id, count
  FROM (SELECT id, COUNT(*) as count
    FROM Shohin
    GROUP BY shohin_bunrui) AS ShohinSum;
```
- AS は省略可能

## スカラ・サブクエリ
WHERE句 の中では集約関数が使えない。ではAVGやSUMの結果を WHERE句 の中で使いたい場合はどうすればいいか

```sql
SELECT id, tanka
FROM shohin
-- WHERE句 の中では集約関数は使えないためエラー
WHERE tanka > AVG(tanka);
```

そんなときはスカラ・サブクエリを利用する
```sql
SELECT id, tanka
FROM shohin
-- 平均単価を求めるスカラ・サブクエリ
WHERE tanka > (SELECT AVG(tanka) FROM shohin);
```

## なんでWHERE句のなかで集約関数が使えないのか
https://qiita.com/k_0120/items/a27ea1fc3b9bddc77fa1
SQLの実行順は次のようになっている
```
FROM句
↓
JOIN句
↓
**WHERE句**
↓
**GROUP BY句**
↓
HAVING句
↓
SELECT句
↓
ORDER BY句
↓
LIMIT句
```
そもそも、グループ化されたデータに対して処理をかけて1つの値にまとめる関数を集約関数とよぶ
GROUP BY句 よりも WHERE句 が先に実行される、つまり WHERE句 が実行されるタイミングではまだグループ化は行われていない。
よって GROUP BY句 が実行されるよりも前に集約関数を使うことができない

## スカラ・サブクエリ をかける場所
- スカラ・サブクエリは基本的にはスカラ値をかけるところにはどこでもかける。

# 相関サブクエリ
- 小分けにしたグループ内での比較をするときにつかう
- GROUP BY と同じく相関サブクエリも集合の「カット」という機能を持っている
- 相関サブクエリの結合条件はサブクエリの中に書かないとエラーになってしまう

- GROUP BY でカットしたグループごとに集計を適用し、その結果を WHERE句に使いたい場合につかったりする。

## 具体的な例
その商品分類の販売単価よりも高い販売単価の商品を出力したい際に
```sql
select id, hanbai_tanka
from shohin
where hanbai_tanka > (select AVG(hanbai_tanka)
  from shohin
  group by shohin_bunrui);
```

とするとエラーになる。(スカラサブクエリを書くべきところに、3行の結果を返すクエリを記述してしまっているため)
相関サブクエリを使うとうまくかける

```sql
select id, hanbai_tanka
from shohin as s1
where hanbai_tanka > (select AVG(hanbai_tanka)
  from shohin as s2
  where s1.shohin_bunrui = s2.shohin_bunrui
  group by shohin_bunrui);
```

このように、相関サブクエリはテーブル全体ではなく、テーブルの一部のレコード集合に限定した比較をしたいときに使う

# 組み込み関数など
## ABS
絶対値を返す
NULL を引数にとると NULL を返す

# 変換関数
- 型変換・値の変換など少し特殊な働きをする関数群

## COALESCE
- 「コアレス」「コォアリース」とかって読むらしい
- 可変個の引数をとり、最初にNULLでない値を返す
- 演算の途中に NULL が紛れ込むと結果が全て NULL になってしまうのを防ぐために使ったりする

```sql
select COALESCE(str2, 'NULLだよ') from SampleStr;
```

# 述語
- 述語とは、戻り値が真理値になる関数のこと

## LIKE
- 部分一致検索(前方一致、中間一致、後方一致)を行うことができる

前方一致はこう書く
```sql
select str
from tableName
where str LIKE 'hoge%';
```
中間一致: `'%hoge%'`
後方一致: `'%hoge'`

`%` は0文字以上の任意の文字列を表す
`_` は任意の1文字を表す

## BETWEEN
- 範囲検索を行うことができる
- `WHERE col BETWEEN 10 AND 20` と書いた場合、10"以上"20"以下"の値が返される(両端の値を含むことに注意する)

## IN
- 同一のカラムに対する `OR` 条件がたくさんある場合、`IN` を使うとスッキリかける

```sql
select id, age
from user
where age = 19
  or age = 20
  or age = 21;
```
これを `IN` を使って書き直すとこう書ける

```sql
select id, age
from user
where age IN (19, 20, 21);
```

- 含まない行を検索したい場合は `NOT IN` を使う

### IN の引数にサブクエリを指定する
IN の引数には以下のようにサブクエリを指定することができる
where句の条件を動的に変えたい場合はこの方法を使うのがよさそう

```sql
select shohin_id, hanbai_tanka
from shohin
where shohin_id IN (
  select shohin_id
  from tenposhohin
  where tenpo_id = '000c'
);
```

## EXISTS
- 基本的には `IN` で書き換えることができる
- `IN` の方が直感的なので `IN` で代用できる場面ではそうする方がわかりやすいことが多い

↑の select文を `EXISTS` を使って書き換えると以下のようになる

```sql
select shohin_id, hanbai_tanka
from shohin as s
where EXISTS (
  select *
  from tenposhohin as t
  where tenpo_id = '000c'
  AND t.shohin_id = s.shohin_id
);
```

# CASE式
- CASE式には単純CASE式と検索CASE式の2種類がある。検索CASE式は単純CASE式の機能をすべて含む
- 条件分岐を記述することができる
- (MySQL だと `||` で文字列結合ができないので注意(`CONCAT('a', 'b')`のようにかく))

## CASE式による行列変換
```sql
select 
  (SUM(
    CASE
      WHEN shohin_bunrui = '衣服' 
      THEN hanbai_tanka
      ELSE 0
      END
  )) as sum_a,
  (SUM(
    CASE
      WHEN shohin_bunrui = 'キッチン用品' 
      THEN hanbai_tanka
      ELSE 0
      END
  )) as sum_b
from
  Shohin;
```

| sum_tanka_ihuku | sum_tanka_kitchen | sum_tanka_jimu |
|:-:|:-:|:-:|
| 5000 | 11180 |600 |

## 検索CASE式 と 単純CASE式 の違い
単純CASE式は比較対象が固定される
読みやすいし記述量も少ないのでできるだけ単純CASE式を使うのがよさそうなきがする

```sql
-- 検索CASE式
select 
    CASE
      WHEN shohin_bunrui = '衣服' THEN CONCAT('A: ', shohin_mei)
      WHEN shohin_bunrui = 'キッチン用品' THEN CONCAT('B: ', shohin_mei)
      WHEN shohin_bunrui = '事務用品' THEN CONCAT('C: ', shohin_mei)
      ELSE 0
      END
from
  Shohin;

-- 単純CASE式
select 
    CASE shohin_bunrui
      WHEN '衣服' THEN CONCAT('A: ', shohin_mei)
      WHEN 'キッチン用品' THEN CONCAT('B: ', shohin_mei)
      WHEN '事務用品' THEN CONCAT('C: ', shohin_mei)
      ELSE NULL
      END
from
  Shohin;
```

## NOT IN で指定したリストやサブクエリーの値に NULL が存在すると、常に空の結果セットが返ってくる
理由は NOT IN で指定された全てと合致しない行のみが返されるが、 NULL との比較は常にUnknown が返るため

# テーブルの足し算と引き算
- 集合演算子は重複行を削除する(ALL オプションをつけた時を除く)

## 集合演算の注意事項
- 演算対象となるレコードの列数は同じであること
- 演算対象となるレコードの型が一致していること
- order by は最後に1つしかつけられない

## テーブルの共通部分の選択 INTERSECT
- mysql ではサポートされていない

```sql
SELECT shohin_id, shohin_mei
  FROM Shohin
INTERSECT
SELECT shohin_id, shohin_mei
  FROM Shohin2
ORDER BY shohin_id;
```

# テーブルの結合
## INNER JOIN
一致したレコードだけ出力する

## OUTER JOIN
どちらかのテーブルに値が存在していれば出力する

## CROSS JOIN
直積
すべての列の掛け合わせ
