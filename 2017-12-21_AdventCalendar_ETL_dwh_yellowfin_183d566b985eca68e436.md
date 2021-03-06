<!--
title:   業績を見える化&自動化したら意思決定スピードも給与も上がった話
tags:    AdventCalendar=2017,ETL,dwh,yellowfin,読み物
id:      183d566b985eca68e436
private: false
-->
[エイチームライフスタイルアドベントカレンダー2017](https://qiita.com/advent-calendar/2017/ateam-lifestyle)、残り数日となりました。そして、2018年も残りわずか、いよいよ年の瀬が感じられる頃になってきましたね。
本日は、[株式会社エイチームライフスタイル](http://life.a-tm.co.jp/)でエンジニアをしております、@aiji42 が、「業績を見える化&自動化したら意思決定スピードも給与も上がった話」と題して、今年前半に取り組んでいた、経営的指標見える化&集計自動化プロジェクトの話をお伝えします。よろしくお願いします。

## ざっくり言うと
本記事でお伝えしたいのは、会社の業績管理をつかさどる、データウェアハウスの構築と、BIツール選定・導入のお話になります。
技術的なお話ではなく、システム構築において、各フェーズで注意や意識をすべき点に関して記載しています。
タイトルに惹かれてここまでやってきた方は、ぜひ最後までお読み下さい。

+ [設計とツールの選定で気をつけること](#設計とツールの選定で気をつけること)
    + [●ビジネスモデルを理解せよ](#ビジネスモデルを理解せよ)
    + [●データの持ち方と処理は切り離して考えよ](#データの持ち方と処理は切り離して考えよ)
    + [●BIツールの選定基準](#biツールの選定基準)
+ [開発中に気をつけること](#開発中に気をつけること)
    + [●ExcelとSQLをマスターせよ](#excelとsqlをマスターせよ)
    + [●Windows PC を用意せよ](#windows-pc-を用意せよ)
    + [●集計担当者を巻き込んでテストせよ](#集計担当者を巻き込んでテストせよ)
+ [運用開始でつまずかないために](#運用開始でつまずかないために)
    + [●完全にExcel集計をやめるということはしない](#完全にexcel集計をやめるということはしない)

## はじめに
皆さんの会社では、業績や経営的指標をどんな方法で管理されていますか？

1年前の話になりますが、私が所属する事業部では、CV件数、売上、広告費、CPA、ARPUなど様々な数字が、複数のExcelで管理されておりました。
各集客手法の担当者が、各自でExcelファイルを作成し、2日に1度集計をして、事業全体の数字を管理するファイルにマージするというものです。

![スクリーンショット 2017-12-21 11.56.58.png](https://qiita-image-store.s3.amazonaws.com/0/46467/98c9c36d-c239-4edc-b905-58b09ddf96ae.png)

このようなやり方で管理をされている会社は、意外に多いのではないかと推測します。

### **数字に対しての文化や理想と、現実のギャップ**
エイチームグループでは、全業績をオープンにしており、業種や役職に関係なく、全員が自主的に数字を見ているという文化がある一方で、下記のような問題も多く存在しておりました。

**1. 担当者が有給や出張で不在になると、全体の集計も遅れて速報性が失われる**
**2. 各担当でフォーマットが異なり、他者がサポートしにくい**
**3. どのファイルが正しく、最新のデータなのか、担当者に確認しないとわからない**
**4. データの汎用性がなく、成長推移を見たり対比し難い**
**5. 異動等に伴う、引き継ぎコストが甚大**

このような問題があるため、集計する側にも負荷がかかりますが、数字を確認して意思決定したり、施策のPDCAを回す側にも無駄なコストが発生していました。

エイチームライフスタイルでは今年の初めに[コミットフライデー](http://life.a-tm.co.jp/news/corporate-002/)という制度を導入しております。働き方改革と世間では騒がれておりますが、弊社でもより一層、業務の効率化の必要性が一気に高まりました。

### **理想の姿と自動化プロジェクトの発足**
これらの課題を踏まえて、目指すべき理想の形は、

**集計作業コストをゼロにし、リアルタイムかつ容易に数字が確認できるようにすることで、生産性を向上させる**

この一言に集約されます。

一方で、Excel集計だからこそできていた良いことも、いくつかありました。

**1. 担当者が自由に操作でき、ビジネスの変化に容易に対応ができる**
**2. 手動で数字を更新しているからこそ、担当者は僅かな変化や問題にも気づける**

１に関してはシステム側の設計でカバーが出来ますが、２に関しては完全自動化という行動と相反しています。

これら、良かったところは残しつつ、問題を解決して、理想の姿に近づくため、プロジェクトチームが立ち上がりました。

本記事は、そのプロジェクトの中で、奮闘したことや、思考を凝らしたことを書いていきたいと思います。

## 設計とツールの選定で気をつけること

まずは設計からです。
必要に応じてツール等も導入することも必要になります。弊社では最終的なアウトプットにBIツールを導入し、それ以外のデータウェアハウス(DWH)やデータに対して処理を加える[ETL](https://ja.wikipedia.org/wiki/Extract/Transform/Load)は内製しました。
また、売上や費用の単価となるマスタ情報もすべてExcelで管理されていたので、単価マスタ専用のデータベースとインタフェースも作成しました。

![スクリーンショット 2017-12-21 11.56.50.png](https://qiita-image-store.s3.amazonaws.com/0/46467/19e9c53b-d3a9-5a3e-3b6a-0cd266516b7b.png)


### **●ビジネスモデルを理解せよ**
こちらは、事業やサービスに寄り添うエンジニアが業務を進める上で当たり前のことです。
ですが、売上に当たるお金の経路の概要は理解していても、それがどのように計上され、どのタイミングでどこに請求しているか、など、細かいところまで理解しているエンジニアは少ないのではないでしょうか。
リスティングやディスプレイ広告など、宣伝活動を行っているのであれば、それにかかる広告宣伝費の支払先や、どのような形体で計上されているかなども重要な要素です。

会計管理ツールを作るわけではないので、すべてを網羅する必要はありませんが、下記のような内容は最低限把握し、仕様に落とすべきかと思います。

+ **単価を集計するための期間がいつからいつまでなのか**
    + 月初から月末
    + 毎月○日などの締日がある
+ **CVやクリックに対して売上/費用が計上されるのはいつなのか**
    + 当月に計上
    + 翌月に計上
+ **単価が変動する条件**
    + 時期(営業交渉によって毎月変動するなど)
    + 件数に応じたテーブル制
    + 固定と従量
+ **無効となる条件**
    + 送客後ユーザが○○したら件数としてカウントしない
    + リスティングの無駄クリック補正

### **●データの持ち方と処理は切り離して考えよ**
下記は私が参考にした(したらよかったものも含む)データウェアハウス(DWH)の基本設計です。

+ [データウェアハウスアーキテクチャ(1)](http://www.dbstories.com/entry/datawarehouse_architecture_01)
+ [DWH: スタースキーマをベースにあらためて考えてみたデータモデリングの９つのこと](http://crmprogrammer38.hatenablog.com/entry/2017/02/21/155943)

大切なのは、DWHの設計は、**基本形を押さえ、最終的なアウトプットに依存しない設計にすること**です。
設計と同時に、アウトプットになるBIツールも選定することになりますが、そことは切り離して考えなければなりません。

### **●BIツールの選定基準**
弊社では下記のような基準で選定を実施しました。

#### **料金**
[Redash](https://redash.io)のようなOSSを利用する場合、無料で導入できますが、自社でのメンテナンスコストが上がりますし、ローカライズされてない事が多いため、エンジニア以外のユーザは慣れるまでのハードルが高い傾向があります。

一方で、[Tableau](https://www.tableau.com/ja-jp)や[DOMO](https://www.domo.com/jp)、[Yellowfin](https://yellowfin.co.jp)などの有料ツールを利用する場合は、利用料がかかりますが、その分サポートを受けることができ、日本語対応されているものが多く、導入ハードルも比較的低いです。

その上で有料のものを選ぶ場合には、**1ユーザあたりいくらになるか、年額/月額ではどうか**を比較する必要があります。

#### **ツールがどこで稼働しているか**
ツールが稼働しているサーバがどこにあるかも重要です。
**自社の管理配下で稼働させられるのか、ベンダーの管理配下での稼働になるか**

こちらは、会社や組織が定める情報管理のポリシーに従うことが重要です。
弊社では、業績データは最重要機密情報と考えているため、自社管理ネットワーク内で運用できるということが、ツールの選定において大きな比重を占めました。

#### **ETL機能を内包しているかどうか**
**BIツール本体がETL機能をもつか**も重要です。*[ETL - (Wikipedia)](https://ja.wikipedia.org/wiki/Extract/Transform/Load)*
例えば、TableauやDOMOはETL機能を内包しています(私の解釈になりますが、その分、利用料が高い印象です)
BIツール側でカバーできれば、導入コストは大きく削減できますが、ベンダーロックインのデメリットも発生します。

弊社では、ETLはビジネスロジックやスケールに応じて大きく変わる可能性があるので、拡張性を担保するために内製という選択肢をとりました。そのため、BIツールにはViewとしての表示機能のみ委任しています。

#### **BIツールの表現力**
折れ線グラフ、円グラフ、ロウソク足、ヒートマップなどさまざまな表現方法がありますが、事前にどんなデータをどのような形で表現するのか、何パターンか想定し、その表現が実現できるかを検証するとよいでしょう。

弊社では、元々Excelで数字管理をしていたこともあり、表での表現力も重要視しています。
ツールによってはリッチでグラフィカルな表現手法に力を入れていて、意外に表での表現力が乏しいものもあるので、かっこいい見た目ばかりに気を取られていはいけません。
表での描画に関しては、下記の事項を重要視しました。

+ **三桁でのカンマ区切り**
+ **小数点の桁数指定**
+ **単位の付与**
+ **条件付き書式** (文字色/太さ/セルカラー/アイコンなど)

## 開発中に気をつけること
基本的にはソフトウェアやサービスの開発で気をつけるべきことや、意識すべきこととはあまり違いはありません。
本記事では、集計自動化の開発に特化した内容を記載します。

### **●ExcelとSQLをマスターせよ**
本プロジェクトの開発はデータとの戦いになります。
特にETLの開発は、既存のExcelで処理されていることをSQLに変換していく作業です。また、BIツール上での表現もSQLを書くことが基本となります。
その為、**Excelのスキルと、SQLの理解はこのプロジェクトのスピードに大きく影響します。**

ちなみに、弊社では開発の大半をSQLを記述する作業で占めており、最終的にETL部分は 4,000行強のSQLを記述しました。(この大量のSQLをどう管理すべきかに関しては別の機会に書きたいと思います。)

### **●Windows PC を用意せよ**
Macで開発しているのであれば、Windows PC をサブ機として用意することを推奨します。
前述の通りETLの開発は、Excelでの処理をSQLに変換していく作業が大半になりますので、Excelがスムーズに立ち上げられなければ、大きなストレスとなります。

残念なことに、同じデータ量を扱うにしても、Mac版のExcelはWindowsに比べて異常なほどにメモリを消費し、Windowsでは数秒で終わる処理が、Macでは数十秒かかるということが多くあります。

### **●集計担当者を巻き込んでテストせよ**
こちらは、後の運用をスムーズに行う事が大きな目的になっております。

エンジニア以外のメンバーにとって、データの入力インターフェイスと、最終的なBIツール上での描画の間は全てブラックボックスになります。
システム化することで、これまでExcelで見えていたところが、すべて不透明になるのですから、これは混乱の原因になります。
この問題を解消するために、**テストの段階から参画してもらうことで、担当者はインプットとアウトプットの関係性のイメージがつかみやすくなり、導入を円滑に進めることができます。**

## 運用開始でつまずかないために
どんなにテストを積み、不具合がないことを確認していても、実運用を開始した際に数値がおかしいということがよくありました。
私の経験上、実装したロジックの欠陥よりも、入力されたマスタデータに問題があることを先に疑うと、問題解決にたどり着きやすくなります。

### **●完全にExcel集計をやめるということはしない**
通常であれば、システム導入後、一定期間の並行運用(Excelとシステム)期間を設け、その後完全に移管するというのがベストに思えます。
しかし弊社では、Excelの集計頻度は落としましたが、完全にExcelでの集計をやめてはいません。
それは下記のような理由からです。

#### **不具合の早期発見とリスクヘッジのため**
集計が自動化され効率化したことにより、数字の信頼性に対してのニーズと関心も高まりました。

業績は非常に重要なデータであり、それをもとに様々な意思決定がなされます。
そのため、月中のデータに間違いがあり、月末に会計を締めたときに問題に気づいたでは遅すぎると同時に、大きな損失を生んでいる可能性もあります。

このリスクヘッジのために、Excelでの集計は週に1回程度で継続し、データを突き合わせて問題がないことを検証しています。

#### **数字に対しての意識を維持するため**
あまり論理的ではないかもしれませんが、私は手動で集計を行うとで数字に対しての意識や執着心の向上にもつながると考えています。
自動化したことによって、中身がブラックボックスになれば、それだけ数字に対しての心理的距離は大きくなると思います。

この問題を危惧し、少なくとも運用担当者は自身の管理する数字に対して意識を高く持って欲しいという思いから、頻度は週1回に落としましたが、手動での集計を担当者にはお願いしています。

## まとめ
+ [設計とツールの選定で気をつけること](#設計とツールの選定で気をつけること)
    + [●ビジネスモデルを理解せよ](#ビジネスモデルを理解せよ)
    + [●データの持ち方と処理は切り離して考えよ](#データの持ち方と処理は切り離して考えよ)
    + [●BIツールの選定基準](#biツールの選定基準)
+ [開発中に気をつけること](#開発中に気をつけること)
    + [●ExcelとSQLをマスターせよ](#excelとsqlをマスターせよ)
    + [●Windows PC を用意せよ](#windows-pc-を用意せよ)
    + [●集計担当者を巻き込んでテストせよ](#集計担当者を巻き込んでテストせよ)
+ [運用開始でつまずかないために](#運用開始でつまずかないために)
    + [●完全にExcel集計をやめるということはしない](#完全にexcel集計をやめるということはしない)

## 最後に
私はこのプロジェクトを通じて非常に多くのことを学びました。
今では、エンジニアでありながら、事業部の誰よりも数字に関して理解があると自負しております。
これは事業メンバーからの信頼につながります。

また、Before/Afterで効果を見た時に、私の所属している事業部では `30分/回 × 3回/週 × 担当者5人 = 約7.5時間/週` での**工数が3分の1に削減できました。**(前述の通り完全に手動集計をやめた訳ではありませんのでゼロにはなりません)
これは年間に直せば約270時間の削減になります。

そして、それ以上に多くのリターンがありました。

例えば、
**マーケティングや営業的な意思決定はすべて毎日の朝会の中で完結できるようになりました。**
これまで集計は、2日に1回の頻度で行われていましたので、最新のデータが確認できないデッドタイムが、2日以上(休日を挟めば3日以上)発生していました。
それが、毎日出社前に集計が自動で完了されるのですから、朝会では直近のデータを確認しながら議論を進めることが可能となります。
その場で意思決定ができるわけですから、短いサイクルでPDCAを回すことが可能になります。2日以上あったデッドタイムが1日に短縮されました。
もちろん、朝会以外のミーティングは不要になりますので、業務改善にもつながりました。

そして、上記のような功績がみとめられ、給与アップという効果も(笑)

ちなみに、弊社が導入したBIツールのベンダーである[Yellowfin社](https://yellowfin.co.jp/)に、導入事例としてインタビューして頂いた記事が、Yellowfin社のブログに掲載されていますので、そちらも併せてお読みいただけますと幸いです。
[【Yellowfin導入事例】株式会社エイチームライフスタイル](https://yellowfin.co.jp/blog/2017/11/10_casestudy)

本記事が、今後私と同じような活動をされる方のお役に立てますと幸いです。

[エイチームライフスタイルアドベントカレンダー2017](https://qiita.com/advent-calendar/2017/ateam-lifestyle)、明日はエイチームライフスタイルのエンジニアを牽引するシニアエンジニア @tsutorm が「AlexaのCustom Skills入門(仮)」を書いてくれるらしいので、お楽しみに。

[株式会社エイチームライフスタイル](http://life.a-tm.co.jp/)では、一緒に働けるチャレンジ精神旺盛な仲間を募集しています。興味を持たれた方はぜひエイチームグループ採用サイトを御覧ください。
http://www.a-tm.co.jp/recruit/