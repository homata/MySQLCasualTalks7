


#[fit] Continuous Restoreへの道



---

#[fit]who are you

---

![](http://stat.ameba.jp/user_images/20141211/06/kakerukaeru/84/f8/p/o0354033513155423588.png)

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

# [fit]game

![](http://stat.ameba.jp/user_images/20131120/16/girlfriend-kari/18/1a/j/o0640068012755342735.jpg)

---

# [fit]pigg

![](https://stat.brave.pigg.ameba.jp/img/top/bg_header2.png)

---

# [fit]blog

![](http://stat100.ameba.jp/p_skin/w_officialskin/ab-anna06/img/header.jpg)

---

#[fit]Community

![](http://stat100.ameba.jp/common_style/img/ameba/lp/girls-talk/title_h1.jpg)

---

# [fit]MySQLいっぱい使ってるよ

![](http://stat.ameba.jp/user_images/20141202/19/principia-ca/61/a0/j/o0800053213147364213.jpg)

---


#[fit]agenda

---

- 継続的りすとあやってみた
- どう実現したか
	- Backupマデ
	- Restoreマデ
- もうちょっと考える
- 使ってみた所感
- 今後

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

![](http://stat.ameba.jp/user_images/20141211/05/kakerukaeru/92/25/j/o0450031613155415822.jpg)


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

# 確認項目、何があるかなー

- テーブル破損
- テーブルの中身
- テーブル定義
- index確認

```bash
    mysqlcheck --all-databases
	CHECKSUM TABLE hogehoge
	SHOW CREATE TABLE hogehoge
    SHOW INDEXES FROM hogehoge
```

---

# ただ

---

# 毎回checksumを取るのは正直重い

---

# ので、
# ２つのレベルで考える

---

- 都度check
    - テーブル破損
- 低頻度check
    - テーブル破損
    - テーブルの中身
    - テーブル定義
    - index確認

---

# [fit]都度checkの流れ

---

![fit](https://cacoo.com/diagrams/wAi5Qle3OcD5kBPG-4546C.png)

---

# 都度check

### Restoreの流れの最期にmysqlcheckをはさんだだけ
### 一旦テーブルが壊れてなかったらいいよね

![fit](https://cacoo.com/diagrams/wAi5Qle3OcD5kBPG-4546C.png)


---

# [fit]低頻度checkの流れ

---

![fit](https://cacoo.com/diagrams/5wRmkoXqhlq5ESwY-4546C.png)

---

# なんのこっちゃ

![fit](https://cacoo.com/diagrams/5wRmkoXqhlq5ESwY-4546C.png)

---

- stop Replication
    - pre_checksum
        - checksum table,show create table,show indexes from
    - backup
- start Replication
    - restore
    - diff pre_checksum
    - mysqlcheck

![fit](https://cacoo.com/diagrams/5wRmkoXqhlq5ESwY-4546C.png)

---

#これでdataの正当性も
#ある程度担保できたかな

---

#[fit]：TODO
#[fit]jobのスクショ貼る
#[fit]わざと更新させて、diff prechecksumをコケさせる

---

# 動いてるね

---

# 安心して寝れる
# ( ˘ω˘ ) ｽﾔｧ…

---

#[fit]使ってみた所感

---

#[fit] ~~正直今週稼働し始めたばっかだからまだ感想ない~~
うちだと結構なService数が立ち上がってるんだけど、
１Serviceずつrestore出来るかの確認を心温まる手作業で確認しなくても良くなったという精神的な安らぎを得ました。
<br />
あと、Restoreにどれぐらいの時間がかかるかとか分かったりしてちょっと楽しい

---

#[fit]今後

---

#不安点、改善点

- check項目もっと詰めれる気がする
- まだ５Serviceぐらいでしか稼働してない
    - もっと実績増やす
- まだ100GBぐらいまでの容量のrestoreしかしてない
    - 容量増えてきたら今の方法だと
    checkが終わらない気はしてる

---

#[fit]まとめ

---

#[fit]Backup担保により
#[fit]快適なDB破壊ライフを！＾ｐ＾

---

#[fit]ご静聴ありがとうございました