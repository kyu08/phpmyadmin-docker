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
- AS は省略可能
```sql
SELECT id, count
  FROM (SELECT id, COUNT(*) as count
    FROM Shohin
    GROUP BY shohin_bunrui) AS ShohinSum;
```

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
