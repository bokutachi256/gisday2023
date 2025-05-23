# はじめに

本コースではPythonで作成したマルチエージェントシミュレーションを行い，結果をArcGIS Proで図化する事をゴールとしています．マルチエージェントシミュレーションは個々のエージェントがそれぞれのルールによって行動し，お互いに影響し合って全体の振る舞いをシミュレートする方法です．今回のワークショップでは歩行により避難行動を行う避難者を避難者エージェントとし，道路部分にランダムに配置した避難者エージェントがゴール地点である避難所に到達するまでをシミュレートします．避難者エージェントは空間全体を区切ったメッシュに一つだけしか入れないようになっているため，混雑する場所などでは渋滞が発生して避難所までの所要時間が複雑に変化します．混雑する場所をヒートマップで表示することにより，行動のボトルネックとなる場所を知ることができます．また，シミュレーション全体のアニメーションも作成できます．

本ワークショップではマルチエージェントシミュレーションのプログラム自体にはほとんど触れません．プログラム中にはコメントがたくさん書いてあるのでそちらを参照してください．また，Pythonによるマルチエージェントシミュレーションのプログラム作成に関しては，GIS Day in 東京 2021 Eコース「Pythonで作るマルチエージェントシミュレーション」リポジトリ（[https://github.com/bokutachi256/gisday2021](https://github.com/bokutachi256/gisday2021)）を参照してください．

# サンプルデータの準備

## サンプルデータのダウンロードとコピー

本コースのgithubのページ（[https://github.com/bokutachi256/gisday2023](https://github.com/bokutachi256/gisday2023)）を開きます．

![githubのページ](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231113_214533.png)

`sample_data`フォルダにはワークショップで使用するサンプルデータがあります．サンプルデータは以下の4ファイルです．

* `aoi.geojson`: 対象地域を囲んだ四角形のポリゴン
* `blda.geojson`: 対象地域の建物輪郭線ポリゴン
* `road_polygons.geojson`: 対象地域の道路ポリゴン
* `wa.geojson`: 対象地域の水域界ポリゴン

上記の建物輪郭線ポリゴン，道路ポリゴン，水域界ポリゴンは[基盤地図情報](https://www.gsi.go.jp/kiban/)の基本項目データを加工して使用しています．

ファイル形式はGeoJSONです．GeoJSONはオープンな空間情報ファイルフォーマットで，PythonのGeoPandasモジュールで簡単に使用できるほか，ArcGISやQGISなどのデスクトップGISでも使用することができます．

![サンプルデータのリスト](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231113_214816.png)  

サンプルデータをクリックするとデータのプレビュー画面に移動します．画面の右上にあるダウンロードボタンを押して，4個のデータすべてをダウンロードししてください．

![サンプルデータのダウンロード](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231113_215205.png)  

ダウンロードが完了したらスタートメニューからArcGIS Proを開き，新しいプロジェクトから「マップ」をクリックし，`GISDAY2023`を作ります．プロジェクトの場所はデフォルトで構いません．「このプロジェクトのための新しいフォルダーを作成」にはチェックを入れてください．ワークショップ終了後はプロジェクトフォルダごとUSBメモリなどにコピーすれば持ち帰りできます．

![プロジェクトの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231113_220134.png)  

次にダウンロードしたサンプルデータをプロジェクトフォルダにコピーします．カタログウインドウからフォルダーを開いてGISDAY2023フォルダを右クリックし，ファイルエクスプローラーで表示を選びます．ファイルエクスプローラーでプロジェクタフォルダが開きますので，ダウンロードした4個のファイル（`aoi.geojson`，`blda.geojson`，`road_polygons.geojson`，`wa.geojson`）をプロジェクトフォルダにコピーします．

![サンプルデータのプロジェクトフォルダへのコピー](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_131312.png)  

ArcGIS ProはGeoJSONを直接開けないので，サンプルデータをファイルジオデータベースにコピーします．ファイルジオデータベースはArcGIS Proの標準的なデータフォーマットで，シェープファイルとは違ってデータベースの構造を持っているために大量のデータでも高速に処理ができます．ジオデータベースについての詳細はESRIジャパン社のホームページ（[https://www.esrij.com/gis-guide/esri-dataformat/gdb-overview/](https://www.esrij.com/gis-guide/esri-dataformat/gdb-overview/)）をご覧下さい．

GeoJSONをファイルジオデータベースにピーするには，ジオプロセッシングから「JSON→フィーチャ」を選び，入力JSONにプロジェクトフォルダーにコピーしたサンプルデータを指定します．出力フィーチャクラスに`GISDAY2023.gdb`を指定し，サンプルデータのファイル名を入力します．ジオメトリタイプはポリゴンのままにします．他の3個のサンプルデータも同様にファイルジオデータベースファイルにコピーします．

![GeoJSONのインポート](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_131914.png)  

次に背景地図に地理院タイルの淡色地図を表示します．地理院タイルは国土地理院によりホスティングされている地図画像配信サービスです．画像のURLを指定するだけで地理院地図の画像を背景として使用できます．地理院タイルについては国土地理院のホームページ（[https://maps.gsi.go.jp/development/ichiran.html](https://maps.gsi.go.jp/development/ichiran.html)）をご覧下さい．

「マップ」メニューの「データの追加」から「パスからのデータ」を選び，パスに淡色地図のURL（`https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png`）を貼り付けます．コンテンツウインドウに「タイルサービスレイヤー」が追加されます．必要であればレイヤー名を「淡色地図」などに変更してください．また，「タイルレイヤー」メニューから「レイヤーのブレンド」を`輝度`にすれば，淡色地図をモノクロ表示にできます．

![地理院タイルの追加](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_132403.png)  

# データの作成

本プログラムにおける避難者エージェントの基本行動ルールを簡単に説明します．

1. 下図のように道路ラスタとコスト距離ラスタがあるとしあます．
2. 道路ラスタのうち値が0のメッシュは避難者エージェントが入れないメッシュ，1のメッシュは避難者エージェントが入れるメッシュです．
3. 青丸は注目している避難者エージェント，緑丸は他の避難者エージェントです．
4. 注目している避難者エージェントの周囲8メッシュを検索します．
5. この中で他の避難者エージェントがいなく，立ち入ることができるメッシュが移動先の候補になります．
6. 移動先の候補のうち，コスト距離ラスタの値が一番小さいメッシュを移動先にします．
7. 移動先が複数ある場合にはランダムに決定します．
8. 移動先がない場合にはエージェントはその場から動きません．
9. これを繰り返すことで，注目している避難者エージェントは青矢印のルートを移動します．
10. 移動先の候補にゴール地点のメッシュがあった場合にゴールしたと判定します．

![避難者エージェントの基本行動ルール](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231117_141901.png)  

以上の説明のように，本プログラムでは避難者エージェントの初期地点（スタート地点），ゴール地点，道路ラスタ，コスト距離ラスタのデータが必要です．

サンプルデータにはこれらのデータは含まれていないため，本節で作成します．避難者エージェントのスタート地点は道路ポリゴン上にランダムに作成し，必要な属性をフィールド演算で追加します．ゴール地点は対象範囲内に好きな点を4点選んで作成します．道路ラスタとコスト距離ラスタの二つはaoiポリゴンと道路ポリゴンに必要な属性を追加し，ラスタ化して作成します．

## スタート地点とゴール地点の作成

スタート地点は道路ポリゴン内にランダムポイントを生成して作成します．道路ポリゴンの道路部分を選択し，ジオプロセッシングウインドウからランダムポイントの作成を選びます．

![ランダムポイントの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_132711.png)  

出力場所を`GISDAY2023.gdb`，出力ポイントフィーチャクラスを`start_points500`，制限フィーチャクラスを`road_polygons`，ポイント数を`500`，最小距離を`1`メートルにします．実行すると選択された道路ポリゴン内にランダムなポイントが500個作成されます．このポイントをスタート地点にします．

![スタート地点の作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_132941.png)  

ゴール地点のポイントデータを作成します．カタログウインドウの`GISDAY2023.gdb`を右クリックして新規・フィーチャクラスを選びます．新しく作成するフィーチャクラスの名前を`goal_points`，フィーチャクラスタイプを`ポイント`にします．次にフィールドを追加します．追加するフィールド名は`goal_ID`，データタイプは`Short Integer`にします．作成するポイントデータの空間参照は道路ポリゴンと同じ平面直角座標系第2系（JGD2011）にします．作成するフィーチャクラスの指定が終了したら，goal_pointsのポイントツールを使って対象範囲に4個のポイントを作成します．

![ポイントデータの新規作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_133624.png)

作成するポイントはどこでも構いませんが，なるべく対象範囲全体に散らばっている方が良いと思います．例題では阿蘇体育館，はな阿蘇美，阿蘇市総合センター付近，あか牛専門店農家レストラン付近の四ヶ所にしました．

![ゴール地点の選定](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_134134.png)  

`goal_points`の属性テーブルを開き，属性`goal_ID`に通し番号をつけます．

![ゴール番号の入力](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_134249.png)  

ポイントを打ったら完了ボタンを押して変更を確定します．確定したら保存ボタンを押して変更を保存します．

![ポイントの確定と保存](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_135125.png)  

## ラスタファイルの作成

### 道路ラスタの作成

道路ラスターは避難者エージェントが立ち入ることができるメッシュを識別するためのデータです．値が1のメッシュは立入が可能，0のメッシュは立入不可能です．今回はすべてのメッシュが立入可能としますが，建物があるメッシュを立入不可能にしたり，水域を立入不可能にすることもできます．

対象範囲全体が立入可能（メッシュの値が1）なラスタは，値が1の定数ラスターで作成します．ジオプロセッシングから定数ラスターの作成を選び，出力ラスターをGISDAY2023フォルダ内に`road.tif`として作成します．出力セルサイズは`5`，出力範囲は`aoi`レイヤーに一致させます．`aoi`レイヤーに一致させると出力範囲に`aoi`レイヤーの範囲の座標が自動的に入力されますが，座標値が半端になるので5の倍数になるようにキーボードから修正してください．実行するとaoiレイヤーと同じ大きさで値がすべて1のラスターデータが作成されます．

ここで作成したラスタファイルは1メッシュの大きさが5m四方になります．本プログラムでは避難者エージェントは1タイムステップあたり1メッシュ移動します．もっと細かいメッシュサイズのデータを作成することもできますが，メッシュが細かくなるほど1ステップあたりの移動距離が短くなるため，長距離を移動させたい場合には計算ステップ数を多く取る必要があります．計算ステップ数が多くなると計算結果のファイルも大きくなり，結果の図化などに時間がかかることがあるので注意が必要です．

![道路を表す定数ラスタの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_135453.png)  

### コストラスタの作成

コストラスタはそのメッシュを通過するのに必要なコストを表すラスタデータです．コストは通過に必要な時間や労力を表す概念で，通過コスト10のメッシュは通過コスト1のメッシュよりも通過するのに10倍の時間ないしは労力が必要なことを表します．コストラスタそれ自身は本プログラムでは使用しませんが，後述のコスト距離を計算するのに必要になります．今回は道路メッシュ1メッシュ当たりの通過コストを1，道路以外のメッシュを10とします．これにより道路は移動しやすい事を表現します．

まずは道路とそれ以外を判別する属性を作成します．road_polygonsの属性テーブルを開き，`Short`型の属性`road`を追加し，道路部分の値を`1`，道路以外の値を`0`にします．

![道路を識別する属性の作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_135855.png)

![道路を識別する属性](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_140244.png)  

次に通過コストを表す属性を作成します．road_polygonsの属性テーブルから属性条件で選択を選び，`road`が1のポリゴンのみを選択します．フィールド演算を用いて選択されたポリゴンの属性`cost`を`1`にします．選択を反転し，`road`が0のポリゴンの属性`cost`を10にします．これで道路部分が1，道路以外が10の属性ができました．

![通過コストを表す属性の作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_140533.png)  

道路データの属性`cost`をラスタ化します．ジオプロセッシングから「フィーチャ → ラスタ」を選び，入力フィーチャを`road_polygons`，フィールドを`cost`，出力ラスターをgisday2023フォルダの`cost.tif`，出力セルサイズを`5`にして環境タブをクリックします．スナップ対象ラスタを`road.tif`にして実行します．スナップ対象ラスタを指定することにより，対象ラスタとぴったり一致するラスタデータを作成できます．

![コストラスタの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_141129.png)  

道路部分が1，道路以外が10の5mメッシュのラスタデータができました．

![コストラスタ](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_141503.png)  

### コスト距離ラスタの作成

コスト距離は対象範囲のすべてのメッシュからゴール地点までの経路上のコストを合計したもので，コスト距離が一番小さいコスト距離のメッシュをたどっていくことで，最小コストでゴールまで到達できます．コスト距離ラスタの作成にはゴール地点のデータとコストラスタが必要です．

ジオプロセッシングから「コスト距離」を選びます．「入力ラスターまたはフィーチャソースデータ」に`goal_points`，「入力コストラスター」に`cost.tif`，「出力距離ラスター」を`GISDAY2023`フォルダ内に`CostDis.tif`とします．「出力バックリンクラスター」を指定する必要はありません．実行するとコスト距離ラスターが作成されます．

![コスト距離ラスタの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_152238.png)  

コスト距離を見るとゴール地点近くのコスト距離は小さく，遠く離れるに従って大きくなっているのがわかります．また道路部分はコストが小さいためにコスト距離も小さくなっています．避難者エージェントは周囲8点のメッシュのうちコスト距離が最小のメッシュをたどって移動しようとするため，結果的に道路上をゴール地点に向かって移動します．

![コスト距離](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_152558.png)  

### スタート地点への属性付与

避難者エージェントに必要な属性を加えます．必須属性は`cost_ID`で，これはどのコスト距離ラスタに従って移動するかを表します．例題ではコストラスタは一つしかありませんが，コスト距離ラスタの指定は必須です．

スタート地点データ`start_points500`に`Short`型の属性`cost_ID`を追加して保存します．`cost_ID`の値は`0`にします．

![参照するコスト距離を表す属性の作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_152853.png)  

プログラム中でのコスト距離ラスタのファイル指定は以下のようになっています．

```python
cost_files = ['CostDis.tif']
```

ここでコスト距離ラスタのファイル名はリストで指定しています．`cost_ID`の値はコスト距離ラスタファイル名リストのインデックスで指定するため，コスト距離ラスタが一つの場合は`0`になります．

### スタート地点とゴール地点をGeoJSONで書き出す

ArcGISでのデータが完成しました．これらのデータをPythonで扱えるGeoJSONに変換します．

ジオプロセッシングから「フィーチャ → JSON」を選び，`start_points500`と`goal_points`をそれぞれ`start_points500.geojson`と`goal_points.geojson`で書き出します．その際に「GeoJSONに出力」にチェックを入れます．

![GeoJSONでデータを保存する](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_153009.png)  

# GoogleColaboratoryでのプログラム実行

## Googleドライブへのデータコピー

GeoJSONに変換したファイルと作成したラスタファイルをGoogleドライブにアップロードします．Googleドライブにログインし，`gisday2023`フォルダを作成します．

![Googleドライブでの新しいフォルダの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_162030.png)  

ArcGISのカタログウインドウでプロジェクトフォルダを右クリックし，「ファイルエクスプローラーで表示」を選択します．

![ファイルエクスプローラーでプロジェクトフォルダを表示する](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_162127.png)  

プログラム実行に必要なファイル（`CostDist.tif`，`goal_points.geojson`，`road.tif`，`start_points500.geojson`）をGoogleドライブの`gisday2023`フォルダにアップロードします．

![Googleドライブへのデータコピー](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_162311.png)  

GIS Day in 東京 2023 Eコースのリポジトリ（[https://github.com/bokutachi256/gisday2023](https://github.com/bokutachi256/gisday2023)）で`GISDay2023_プログラム.ipynb`をクリックし，プログラムを開きます．開いたプログラムの一番上にある「Open in Colab」ボタンを押してプログラムをGoogle Colaboratoryで開きます．GitHubからGoogle Colaboratoryで開いたプログラムはそのまま実行できますが，変更点を保存することができません．コピーして保存したプログラムなら変更を保存できるので，Colaboratoryのファイルメニューからコピーを作成し，コピーしたプログラムの方を修正・実行します．

## Google Colaboratoryでのプログラム実行
![プログラムをGoogle Colaboratoryで開く](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_162548.png)  

上のセルから順番にセルの左側にある実行ボタンを押して実行します．最初に実行したときには警告が出ますので「このまま実行」を押して継続して下さい．

![Google Colaboratoryの警告](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_163302.png)  

途中でプログラムからGoogleドライブにアクセスするための許可が求められます．「Googleドライブに接続」を押し，接続を許可して下さい．

![Googleドライブへのアクセス許可](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_163400.png)  

Google ColaboratoryがGoogleドライブに接続すると左側のフォルダアイコンでGoogleドライブにアクセスできます．gisday2023フォルダが見えているか確認し，以降のすべてのセルを順次実行して下さい．

![Google ColaboratoryからGoogleドライブへのアクセス](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_163552.png)  

## 実行結果のダウンロード

プログラムの実行が終了するとGoogleドライブに計算結果のファイル（`test_200_model_vars.csv`と`test_200_agent_vars.gpkg`）ができます．この2個のファイルをダウンロードしてArcGISのプロジェクトフォルダにコピーしてください．

![実行結果ファイルのダウンロード](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_163826.png)  

ArcGISのカタログウインドウを右クリックして最新の情報に更新すると`test_200_agent_vars.gpkg`が見えるようになります．

![プロジェクトフォルダへの実行結果のコピー](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_164058.png)  

# 計算結果の表示

Google Colabortoryの計算結果は`test_200_agent_vars.gpkg`と`test_200_model_vars.csv`です．このうち`test_200_model_vars.csv`は単純なcsvで，各ステップにおけるゴールした避難者エージェント数と累計のゴール済みエージェント数です．

`test_200_agent_vars.gpkg`はオープンな地理情報データベースであるGeoPackage型式のデータベースで，すべての計算ステップにおけるすべての避難者エージェントの状態が格納されています．GeoPackage型式のデータはArcGISでも表示できますが，そのままでは時系列の表示がうまくできないのでファイルジオデータベースに変換します．

## 時系列データの設定と表示

計算結果の`test_200_agent_vars.gpkg`をファイルジオデータベースにインポートします．`GISDAY2023.gdb`を右クリックして「複数のフィーチャクラスのインポート」を選びます．入力フィーチャに`test_200_agent_vars.gpkg`の`main_agent_vars`を指定し，出力ジオデータベースが`GISDAY2023.gdb`になっていることを確認して実行ボタンを押します．

![計算結果のファイルジオデータベースへのインポート](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_192252.png)  

インポートが済むと`GISDAY2023.gdb`の中に`main_agent_vars`ができるので，これをマップに読み込ませます．

`main_agent_vars`の属性テーブルを開いて中身を確認します．属性`step`は計算のステップ数，`AgentID`はエージェントの識別子で，エージェントごとに値が異なります．`Pos`はメッシュ上のエージェントの座標です．`Goal`はエージェントがゴール済みかどうかを表す属性で，0はまだゴールしておらず，1はゴール済みです．`Strategy`と`Behavior`は今回は使っていません．`pop`はエージェント一つ当たりの人数で，今回は一律で1にしてあります．`start_move`は避難開始までのカウント数で，今回はプログラム内でランダムに1〜50までの値を振っています．`Time`はシミュレーション開始からの経過時間を実時間で表したもので，2020年1月1日を0:00を起算時刻として1計算ステップ当たり5秒です．エージェントは1ステップで5mメッシュを一つ移動しますので，秒速1m（時速3.6km）相当になります．なお，斜め方向の移動も上下左右方向と同じく1ステップで1メッシュ移動します．

![計算結果の属性テーブル](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_193228.png)  

次に計算結果の時系列設定を行います．レイヤ`main_agent_vars`のプロパティから「時間」を選びます．「レイヤーの時間」は属性`Time`の一つだけなので，`各フィーチャに1つの時間フィールドがあります`を選びます．「時間フィールド」には属性`Time`を指定し，「時間間隔」は`事前定義の時間間隔なし`です．これでOKを押します．

![計算結果の時系列設定](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_193316.png)  

メインメニューの「時間」を押し，左側の時間表示ボタンをオンにします．時間の「間隔」を1ステップ当たりの加算時間である`5秒`に設定し，鍵マークを押して「間隔」を固定します．また，「終了の除外」をオンにします．これで開始時刻の「202/01/01 0:00:00」以降から「2020/1/1 0:00:05」より前の状態がマップに表示されます．マップ上部のスライドバーや【再生」ボタンで，設定した時間間隔ごとのアニメーション表示ができます．

![時間設定パネル](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_194011.png)  

## ヒートマップの表示

避難者エージェントの混雑状況をヒートマップを用いて表します．

`main_agent-vars`レイヤを右クリックしてコピーし，マップに貼り付けて複製します．貼り付けた`main_agent_vars`を右クリックして`シンボル`を表示し，プライマリシンボルを「ヒートマップ」にすれば，エージェント密度によるヒートマップが表示されます．手法は`ダイナミック`にしたほうが良いでしょう．また，「レイヤーのブレンド」を`乗算`にすれば，ヒートマップの下にある地理院タイルが透けて見えるようになります．

![混雑状況のヒートマップ表示](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_194920.png)  

# アニメーションの作成

## タイムラインへのキーフレーム追加

避難者エージェントの動きをアニメーションに出力します．「表示」メニューからアニメーションの「追加」を押すと，画面下部にアニメーションタイムラインが表示されます．アニメーションタイムラインの「最初のキーフレームの追加」を押して最初の時刻の地図をキーフレームに追加します．

![アニメーションの作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_200657.png)  

「アニメーション」メニューの「インポート」から「タイムスライダーのステップ」を選んで，すべてのタイムステップをタイムラインに追加します．

![すべてのタイムステップをタイムラインに追加](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_201312.png)  

## アニメーションプロパティの設定

追加したキーフレームは1フレーム当たりの表示時間が長いため，全体としてかなり長いアニメーションになってしまいます．1フレーム当たりの表示時間を0.1秒に設定してアニメーションの全体を短くします．コントロールキーを押しながら`A`キーを押してタイムラインのすべてのキーフレームを選択し，右クリックから「アニメーションプロパティ」を選びます．

![キーフレームのアニメーションプロパティ表示](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_201515.png) 

「アニメーションプロパティ」の「長さ」を0.1秒にすると1タイムステップ当たりの表示時間が0.1秒になります．

![フレームごとの表示時間を0.1秒に設定する](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_201722.png)  

## アニメーションのエクスポート

次にアニメーションを書き出す設定をします．「エクスポート」の「ムービー」を押して「ムービーのエクスポート」を出します．
まずは「ドラフト」の設定を選び，お試しの動画を作成します．「ムービーのエクスポート」の下部でアニメーションの保存先とファイル名を指定し，メディア型式を`MPEG4ムービー (.mp4)`にします．次に1秒あたりのフレーム数を1フレームあたりの表示時間（0.1秒）に合わせて`10`にします．

エクスポートの設定が終わったらエクスポートボタンを押し，アニメーションを作成してください．完成までには数分かかりますが，できあがったものを確認して問題がなければ解像度の高い（HD1080など）アニメーションも作成しましょう．

![アニメーションのエクスポート](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231116_202140.png)  

# 一歩進んだシミュレーションを行う

## テスト用にエージェント数を減らす

今回は避難者エージェントを500個作成しましたが，テストのために動かすなら500個も必要ありません．500個のエージェントから一定数をサンプリングしてプログラムを動かすことができます．エージェントのサンプリングには，「加重コスト距離・壁・スタート地点・ゴール地点の読み込み」セルのスタート地点のデータを読み込んでいる部分に`.sample(200)`を加えます．これで500個中200個がランダムサンプリングされます．

```python
# これを
start_location = gpd.read_file(base_dir + start_geojson).explode(index_parts = True)\
  .cx[ULX+Xsize: ULX + ((cost_file.RasterXSize - 1) * Xsize)\
  , ULY + ((cost_file.RasterYSize + 1) * Ysize): ULY + Ysize]
```

```python
# このように変更します．
start_location = gpd.read_file(base_dir + start_geojson).explode(index_parts = True)\
  .cx[ULX+Xsize: ULX + ((cost_file.RasterXSize - 1) * Xsize)\
  , ULY + ((cost_file.RasterYSize + 1) * Ysize): ULY + Ysize]\
  .sample(200)
```

## 計算ステップ数を変える

このプログラムでは200ステップの計算を行っていますが，もっと長く計算したいこともあります．そのときは「モデルの実行」セルの`runstep`の数を変更します．ただし，実行結果はすべてのエージェントのすべてのタイムステップごとの状態を格納するので，タイムステップをあまり大きくすると実行結果のファイルが巨大なものになってしまいます．ちなみに計算自体はそれほど時間はかかりません．

```python
# これで200ステップ実行
runstep = 200
```

```python
# これで500ステップ実行
runstep = 500
```

## 実行結果のファイル名を変更する

ステップ数を変えたりコスト距離を変更したときなどに実行結果のファイル名を変える必要があるかもしれません．その場合は「モデルの実行」セルの`outfilename`を変えます．

```python
# 実行結果の頭に"test_200"がつく
outfilename = "test_200"
```

```python
# 実行結果の頭に"test_500"がつく
outfilename = "test_500"
```

## エージェントごとに行動パターンを変える

エージェントはコスト距離に従って移動します．本プログラムは複数のコスト距離に対応しているので，必要なだけコスト距離を作成してプログラムに読み込ませる事ができます．例えばコスト距離のデータが4個（`CostDis0.tif`，`CostDis1.tif`，`CostDis2.tif`，`CostDis3.tif`）あった場合，プログラム中でコスト距離のファイルを指定している部分を変更します．

```python
# これを
cost_files = ['CostDis.tif']
```

```python
# このように変更します
cost_files = [`CostDis0.tif`, `CostDist1.tif`, `CostDist2.tif`, `CostDist3.tif`]
```

避難者エージェントの属性値も変更します．エージェントがどのコスト距離を参照するかはエージェントの属性`cost_ID`に従います．前述のように属性`cost_ID`にはコスト距離リストのインデックスを指定するため，リストの一番最初にある`CostDist0.tif`に従う避難者エージェントは`cost_ID`を`0`に，二番目の`CostDis2.tif`に従う場合は`1`にします．

試しに複数のコスト距離を作成してみましょう．ゴール地点の一つを選択し，ジオプロセッシングからコスト距離を選びます．「入力ラスターまたはフィーチャソースデータ」に`goal_points`を選びます．`goal_points`のうちの一つが選択されているので，「入力に選択が含まれています．処理するレコード: 1」と表示され，コスト距離の処理点としてゴールの1点のみが指定されていることがわかります．この状態でコスト距離を作成すると，選択されているゴールまでのコスト距離が生成されます．他のゴール点についても同様にコスト距離を作成し，出力するファイル名を`CostDis0.tif`，`CostDis1.tif`，`CostDis2.tif`，`CostDis3.tif`とします．

![コスト距離の追加作成](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231117_110140.png)  

次に`start_points500`に`Short`型の属性`cost_ID`を追加します．続いてスタート点ごとに`cost_ID`の値を設定します．コスト距離のファイルは新たに作成した4個を使うので，`cost_ID`の値は`0`から`3`までを入れます．ここではランダムに`0`から`3`までの値を入れます．

フィールド演算を開き，入力テーブルを`stasrt_point500`，フィールド名を`cost_ID`にし，式に`Random()*3`と入力します．これで`cost_ID`は0から3までの整数が入ります．

![避難者エージェントの属性変更](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231117_110840.png)  

`cost_ID`が設定できたら，`start_points500`をGeoJSONで書き出します．ここでは`start_points500_2.geojson`として書き出しました．

![スタート地点をGeoJSONでエクスポートする](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231117_111544.png)  

新たに作成したコスト距離（`CostDis0.tif`，`CostDis1.tif`，`CostDis2.tif`，`CostDis3.tif`）とスタート地点のGeoJSON（`start_points500_2.geojson`）をGoogleドライブの`gisday2023`フォルダにアップロードします．

![追加したコスト距離をGoogleドライブにアップロードする](images/GISDay2023_%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88/20231117_111901.png)  

次にプログラムを修正します．「データファイルの指定」セルのコスト距離の指定の部分を変更します．

```python
# これを
cost_files = ['CostDis.tif']
```

```python
# このように変更します
cost_files = [`CostDis0.tif`, `CostDist1.tif`, `CostDist2.tif`, `CostDist3.tif`]
```

また，スタート地点の指定も変更します．

```python
# これを
start_geojson = 'start_points500.geojson'
```

```python
# このように変更します
start_geojson = 'start_points500_2.geojson'
```

## 通行不可能な場所を設定する

今回のサンプルでは道路データはすべての値が`1`のデータになっているため，すべてのメッシュが通行可能になっています．例えば水域を0とした道路データを作成すれば，エージェントはこの部分を通れません．このプログラムでは道路以外（道路データの値が0の部分）のメッシュに壁エージェントを配置します．壁エージェントも避難者エージェント同様にエージェントの一種です．一つのメッシュにエージェントは一つしか存在できないため，壁エージェントがあるメッシュに避難者エージェントは入れません．これにより道路以外に避難者エージェントは入れなくなります．

ただし，道路データとコスト距離ラスタを連動して作成しないと，コスト距離ラスタに従って移動しようとした先が道路以外になっていたときなどに避難者エージェントが動けなくなってしまうことがあります．これを避けるためには，通れないメッシュのコストを高くしたコストラスタを作成し，これをもとにコスト距離ラスタを作成するなどの対策を取る必要があります．

## 基盤地図情報道路縁データをポリゴンに変換する方法

基盤地図情報の道路縁データはラインデータのためそのままではポリゴンとして使用できませんが，狭い範囲であればQGISを使って以下の方法でポリゴン化が可能です．QGISのバージョンが古いと必要なツールがないかもしれません．参考までに使用したQGISのバージョンは3.34-0です．括弧内はプロセッシングツールボックス内のツール名です．

1. 基盤地図情報の道路縁データを読み込む（zipのままで読み込める）．
2. スクラッチレイヤーにポリゴンを作成し，道路縁を切り取る大体の範囲のポリゴンを作成する．
3. 作成したポリゴンのバウンディングボックスを作成する（ベクタジオメトリ：BBoxを出力）．
4. BBOXで道路縁を切り抜く（ベクタオーバーレイ：clip）．
5. 切り抜いた道路縁をマルチパートにする（ベクタジオメトリ：シングルパートをマルチパートに集約）．
6. BBOXをラインに分解する（ベクタジオメトリ：ポリゴンを線に変換）．
7. ラインに分解したBBOXをセグメントに分解（ベクタジオメトリ：線をセグメントに分解）．
8. セグメントにしたBBOXを少し移動（4辺を1mずつ内側に移動する）して道路縁と交差するようにする（ベクタジオメトリ：オフセット線）．
9. オフセット線をマルチパートにした道路縁を線で分割する（ベクタオーバーレイ：split with lines）．
10. 分割したオフセット線とマルチパート化した道路縁をユニオンする（ベクタオーバーレイ：和集合・union）．
11. unionしたものをポリゴン化する（ベクタジオメトリ：ポリゴン化・polygonize）．
12. 交差点などを修正する．