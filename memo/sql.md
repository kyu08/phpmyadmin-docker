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

3.1
shohin_mei は文字列型なので SUM できない
WHERE は GROUP BY よりも前にかこう

3.2
SELECT shohin_bunrui, sum(hanbai_tanka), sum(shiire_tanka)
FROM `Shohin`
GROUP BY shohin_bunrui
HAVING sum(hanbai_tanka) > 1.5 * sum(shiire_tanka);

3.3 
ORDER BY shiire_tanka, torokubi DESC


