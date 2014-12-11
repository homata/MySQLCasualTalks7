


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

# はい

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

# [fit]Slave全台でdata不整合
# Slave一気に脱落[^1]


[^1]: とあるDelete文発行→mysqld落ちる→mysqld_safe経由で上がる→レプリ再開でDelete文発行→mysqld落ちる→mysqld_safe(ry

---

# もしくは

---


# [fit]おばかちゃんが
# [fit]本番DBでDROP DATABASE

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
#＞　 MySQL群　＜
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

- backup用slave
- backup用server
- backup_serverより
    - 定期xtrabackup
    - 随時binlog_sync

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

#[fit]これでRestoreの担保が出来た

---

# いつなんどきサーバが死んでも大丈夫
# `_(ˇωˇ」∠)_` ｽﾔｧ…

---

# [fit]もうちょっと考える

---

# [fit]何をもってリストア完了とするか

---

#[fit] 空っぽのテーブルが
#[fit] Restoreされても意味ない

---

# [fit]なのでRestoreの正当性？
# [fit]も担保してあげよう

---

# どこまでやるか

---

#[fit]backup用のSlaveと
#[fit]restoreされたDBの整合性を考える
 
---

# つまり？

---

#[fit]Backup前とRestore後の確認項目に
#[fit]diffがないことを確認


---

# 確認項目、何があるかなー

- テーブル破損がない事
- テーブルの中身が一致していること
- テーブル定義が一致していること
- indexが一致していること

```bash
    mysqlcheck --all-databases
	CHECKSUM TABLE hogehoge
	SHOW CREATE TABLE hogehoge
    SHOW INDEXES FROM hogehoge
```

---

# [fit]これを踏まえた上でcheckの流れ

---

#No.1
![fit](https://cacoo.com/diagrams/214J2QEBCs3lpF2N-4546C.png)


---

#No.2
![fit](https://cacoo.com/diagrams/1ltuHzNl9Vqgv2tn-4546C.png)

---

#No.3
![fit](https://cacoo.com/diagrams/7UXmKbmX4zxtKvKF-4546C.png)

---

#No.4
![fit](https://cacoo.com/diagrams/oToBQq5cMKxt53jM-4546C.png)

---

# No.5
![fit](https://cacoo.com/diagrams/Jud4CuRIdlhdnh7n-4546C.png)

---

# No.6
![fit](https://cacoo.com/diagrams/8LnU41hNXe3WUiOl-4546C.png)

---

# No.7
![fit](https://cacoo.com/diagrams/3cUSxYyHq7ojqY5X-4546C.png)

---

# No.8
![fit](https://cacoo.com/diagrams/uuPqbPk0YCpyqlrf-4546C.png)

---

# No.9
![fit](https://cacoo.com/diagrams/XTTz51e36WCfHsnH-4546C.png)

---

- stop Replication
    - pre_checksum
        - checksum table,show create table,show indexes from
    - backup
- start Replication
    - restore
    - diff pre_checksum
    - mysqlcheck

![fit](https://cacoo.com/diagrams/XTTz51e36WCfHsnH-4546C.png)

---

# ただ

---

#[fit]毎回このchecksumを流すと
#[fit]時間がかかりすぎる

---

# ので

---

# [fit]実際の運用では２種類に分けています

---

- 毎回実行
    - テーブル破損がない事
- 週一check
    - テーブル破損がない事
    - テーブルの中身が一致していること
    - テーブル定義が一致していること
    - indexが一致していること

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