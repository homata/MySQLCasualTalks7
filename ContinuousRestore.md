


#[fit] Continuous Restoreへの道



---

#[fit]who are you

---

![](http://img.laughy.jp/3838/default_f2d9d5f7f7055738d23f84dbe2500a96.jpg)

## いわながかける @kakerukaeru
ゆとりインフラ園児にあ  
Amebaスマホプラットフォーム面倒見るマン  
Ameba画像配信マン  
最近良く触ってるデータストアはCassandra  

オエーー!!!!　＿＿_  
　　　 ＿＿_／　　 ヽ  
　　 ／　 ／　／⌒ヽ| 
　　/ (ﾟ)/　／ /  
　 /　　 ﾄ､/｡⌒ヽ。  
　彳　　 ＼＼ﾟ｡∴｡ｏ  
`／　　　　＼＼｡ﾟ｡ｏ  
/　　　　 /⌒＼Ｕ∴)  
　　　　 ｜　　ﾞＵ｜  
　　　　 ｜　　　||  
　　　　　　　　 Ｕ  

---

# about
# [fit]Cyberagent

![](http://stat.ameba.jp/user_images/20141202/19/principia-ca/46/eb/j/o0800053213147331179.jpg)

---

![](/Users/a12638/Pictures/kari.jpeg)
# [fit]game

---

![](/Users/a12638/Dropbox/スクリーンショット/スクリーンショット 2014-12-03 21.29.37.png)

# [fit]pigg

---

![](/Users/a12638/Dropbox/スクリーンショット/スクリーンショット 2014-12-03 21.11.07.png)

# [fit]blog

---

![](http://stat.ameba.jp/user_images/20141202/19/principia-ca/61/a0/j/o0800053213147364213.jpg)

# [fit]MySQLいっぱい使ってるよ

---


#[fit]agenda

---

- 継続的りすとあやってみた
- どう実現したか
	- Backupマデ
	- Restoreマデ
- もうちょっと考える
- 今後どうしたい

---

# 継続的
# りすとあやってみた


---

# みなさま
## 不安になったことはありませんか

---

# 例えば

---

# Slave全台で一斉に
# mysqld
# 無限再起動開始
# Slave脱落[^1]


[^1]: とあるDelete文発行→mysqld落ちる→mysqld_safe経由で上がる→レプリ再開でDelete文発行→mysqld落ちる→mysqld_safe(ry

---

# もしくは

---


# [fit] タイミングが悪く
# 連続でSlave死亡
# マスターシングル構成に

---

# ないか
# (＾ω＾)
# ⊃　⊂

---

# あるとするじゃろ？
# (＾ω＾)
# ⊃　⊂

---

# そんなこんなで

---

#＿人人人人人人＿
#＞　Slave全台　＜
#＞　 突然の死 　＜
#￣Y^Y^Y^Y^Y￣

---

#[fit]こんな事もあろうかと

---

#[fit]やっててよかった
#[fit]BACKUP

--- 

#[fit]でも

---

#[fit]これ本当にrestore出来るの？

---

#[fit]もし出来なかったらどうしよう・・・

---

#[fit]残りはマスターだけ・・・

---

#[fit]稼働中のマスターから
#[fit]データ抜き出す事に・・・

---

# 失敗したらどうしよ
![fit](http://blog-imgs-31.fc2.com/t/h/u/thumiya84bka/200912302025482ff.png)


---

#[fit]こんな不安に悩まされないために

---

#[fit]Backupの正当性？の担保をしておきたい

---

#[fit]でも、手で毎回確認するのは・・・

---

#[fit]自動化しちゃいなyo

![fit](http://stat.ameba.jp/user_images/20141202/19/principia-ca/61/7c/j/o0800053213147350001.jpg)

---


# どう実現したか

---

# Backupまでの流れ

---

![fit](https://cacoo.com/diagrams/dfjBaxd615F7Uc9R-4546C.png)

---

![fit](https://cacoo.com/diagrams/dfjBaxd615F7Uc9R-4546C.png)

# Backup

- backup_job用のslave作成
- backup_data置き場のserver準備
- backup_serverより定期的にjob実行
    - 定期xtrabackup
    - binlog随時sync

---

# リストアまでの流れ

---

![fit](https://cacoo.com/diagrams/30rMrryTpp5FGHaa-4546C.png)

---

![fit](https://cacoo.com/diagrams/30rMrryTpp5FGHaa-4546C.png)

# Restore

- API経由でinstance作成
- chef-soloにてmysql_setup
- Backupサーバより最新のsnapshotとbinlogを転送
- test-serverでrestore処理
- instance削除

---

# これで

---

# いつなんどきサーバが死んでも大丈夫
# `_(ˇωˇ」∠)_` ｽﾔｧ…

---

# [fit]もうちょっと考える

---

# 何をもってリストア完了とするか

---

# リストアが完了してもテーブルの中身が0件だと意味なし

---

#[fit]backup用のSlaveの状態と
#[fit]restore後の整合性を考える
 

---

# どんな項目があるかな

- テーブル破損
	- mysql -e 'show databases;' | grep -v .ssh | grep -v Database | xargs mysqlcheck --databases
- レコードのchecksum
	- for database in `mysql -e 'show databases;' | grep -v .ssh | grep -v Databas` ; do for tables in `mysql -e "SHOW TABLES FROM $database" | grep -v "Tables_in_"` ; do mysql -e "checksum table $database.$tables" ;done ; done
- テーブル定義
    - show create table
- index_check
    - どうやってやろうかな  
- もうないか

---

# 上記のcheckの順番的にはこう

---

![fit](/Users/a12638/Pictures/check_sum.png)

---

![fit](/Users/a12638/Pictures/check_sum.png)

# Backup処理の前にcheck_sumを挟む

- Backup_job直前に事前に決めていたcheckを走らせる
- 結果を保持
- そのまま、restoreを走らせる
- ここの処理では、通常のBackup処理とは別の流れで行わないといけない




