# Solo Makeover（ソロ改造）
以下の dashboard を改造していきます。

![Dull Dashboard](img/dull-dashboard.png)

**前提条件**: まず、この dashboard を import する必要があります。
以下の手順で import してください:

1. 左上のメニューボタン（☰）をクリックし、*Dashboards* をクリックします。
2. Dashboards 画面で *New* ボタンをクリックし、*Import* をクリックします。
3. Import Dashboard 画面の *Import via grafana.com* フィールドに `16413` と入力し、*Load* をクリックします。
4. data source を3つ選択するよう求められます:
    - TestData DB には `TestData DB` を選択してください。
    - Prometheus (Cloud) には `Prometheus (Cloud)` を選択してください。
    - LokiNGINX には `Loki (Cloud)` を選択してください。
    - *Import* をクリックします。

*既存の dashboard には、サービスの RED metrics（リクエストレート、エラー、レイテンシ）、Kubernetes Pod のステータス情報、地域別のエンドユーザーアクティビティなど、すでに有用な情報が含まれています。今回の目的は、dashboard 上の情報をより分かりやすく、視覚的にも魅力的なものに改善することです。*

## 'Error Rates' panel を time series から stat panel に変換する

まず Error Rates panel を編集します。エラーレートが許容範囲内か、危険ゾーンにあるか、社内 Service Level Objective（SLO）に違反しているかを分かりやすく表示できるようにします。

![Error Rate Panel](img/error-rate-panel.png)

1. *Error Rates* panel を編集します（panel のタイトルにカーソルを合わせ、右上の縦3点メニューをクリックし、*Edit* をクリック）
2. Visualization Type を *Time Series*（古いバージョンの Grafana では *Graph*）から *Stat* に切り替えます
3. *Stat styles* の設定:
    * Orientation を Auto から Horizontal に変更します。
    * Color Mode を Value から Background Gradient に変更します。
4. 'Value Mappings' セクションを展開し、*Add value mappings* をクリックします:
    * null が N/A にマッピングされるデフォルト設定がある場合は、ゴミ箱アイコン 🗑 をクリックして削除します。
    * *Add a New Mapping* をクリックし、*Range* を選択します。範囲を 0 〜 1 に設定し、Display Text を *OK* にします。色は Blue に設定します。
    * *Add a New Mapping* をクリックし、*Range* を選択します。範囲を 1 〜 2 に設定し、Display Text を *Service Degraded* にします。色は Yellow に設定します。
    * *Add a New Mapping* をクリックし、*Range* を選択します。範囲を 2 〜 100 に設定し、Display Text を *SLO Violation* にします。色は Orange に設定します。
    * *Update* をクリックします。

    Value mapping の設定は以下のようになります:

    ![Value Mappings](img/value-mappings.png)

5.  Panel Title を *SLO Status(Errors) per Data Center* に変更します
6.  *Save Dashboard* をクリックします
## 'K8s Service Status' panel を table から polystat panel に変換する
この table には、すでに分かっている情報が大量に表示されています。もともとこの table の目的は、各サービスコンテナのステータスが 1（UP）か 0（DOWN）かを表示することでした。今回は *polystat* panel を使って、情報の表示をシンプルにすることを目指します。
1. *K8s Service Status* panel を編集します（panel のタイトルにカーソルを合わせ、縦3点メニューをクリックしてコンテキストメニューを表示し、*Edit* をクリック）
2. Visualization Type を *Table* から *Polystat* に切り替えます

    すべての行が1つの平均値に集約されていることに気づくでしょう。コンテナごとにデータを分けるには、query の *Format* を Table から Time Series に変更します。新しいバージョンの Grafana UI では、このオプションは *Options* の下にあります。

    ![Table to Timeseries](img/table-to-timeseries.png)

3. panel オプションの *Global* 設定グループで、Decimals を `2` から `0` に変更します。*Instant* の値は常に 0 か 1 なので、小数点は不要です。
4. Polygon Border Color を Transparent に変更します。
5. Thresholds で *Add Threshold* をクリックします:
    * デフォルトの threshold は値 0 のまま、Color Mode を *ok* から *critical* に変更します。さらに、色を Red から Orange に変更します。
    * *Add Threshold* をクリックして、2つ目の threshold レベルを追加します。新しい Threshold が表示されます。
    * 2つ目の threshold は値を 1 に設定し、Color Mode を *ok* にします。さらに、色を Green から Blue に変更します。
7. オプション panel の下部で *Add value mappings* をクリックします。
    * Value mapping を追加し、値の条件を 1、display text を *UP* に設定します。
    * 2つ目の Value mapping を追加し、条件を 0、display text を *DOWN* に設定します。
8. Font Family を 'Inter' に変更します。
9. Automate Font Color を OFF にします。これでテキストが黒で表示されるはずです。または、Font Color ボックスを使って好みの色を選択できます。
10. *Save Dashboard* をクリックして、この panel の編集モードを終了します。
panel は以下のような表示になるはずです:

    ![K8s Service Status](img/k8s-service-status.png)

## 'Customer Activity' panel を stat panel から geomap に変換する
この panel はかなり見づらい状態です。このデータは OSS の [Loki](https://grafana.com/oss/loki/)（ログ収集ツール）から取得したもので、各地域からのアクセス数を表しているとのことです。カラフルではありますが、情報を読み取るのが非常に難しいです。visualization を地図に変更しましょう！
1. *Customer Activity* panel を編集します（panel のタイトルにカーソルを合わせ、縦3点メニューをクリックしてコンテキストメニューを表示し、*Edit* をクリック）
2. Visualization Type を *Stat* から *Geomap* に切り替えます
3. 右上の *Search options* ボックスに `basemap layer` と入力します。basemap layer のオプションが表示され、他のオプションは非表示になります。
4. *Basemap layer* で、Layer type を *ArcGIS MapServer* に変更し、Server instance を *World Ocean* に設定します。
![Base layer](img/Base-layer.png)
5. 検索バーの '&#215; Clear' をクリックして、"basemap layer" の検索をクリアします。
6. 重要！ これはある時点のデータを表示するビューなので、query の *Query Type* が Range ではなく __Instant__ になっていることを確認してください。
![Query Type](img/Query-Type.png)
7. 地図上にマーカーを追加します。右上の *Search options* を使って *Map layer* を検索し、Layer 1 の *markers* をクリックします。*geoip_country_code* フィールドで国を参照（lookup）したいと思います。
8. *Location Mode* で *Lookup* をクリックし、Lookup Field で *geoip_country_code* を選択します。これで地図上にデータが表示されるはずです。ただし、まだ完了ではありません！
9. Styles で、Size を *Fixed Value* から *Value #Hits by geolocation* に変更します。
10. Symbol を Circles から Star に変更します。circle.svg のテキストをクリックし、Star を選択して 'Select' をクリックします。
11. Color を *Fixed color* から *Value #Hits by geolocation* に変更します。
12. 青とオレンジの色が地図上で溶け込んでしまうため、もう少し目立たせる必要があります。右上の *Search options* ボックスに `thresholds` と入力します。ベースカラーを Dark Purple に変更します。オレンジの丸をクリックし、Dark Purple を選択してから、ポップアップウィンドウの外側をクリックして閉じます。
14. threshold 10 について、同じ方法で色を Dark Orange に変更します。
15. 3つ目の threshold（値 20）のゴミ箱アイコンをクリックして削除します。
16. *Save Dashboard* をクリックして、この panel の編集モードを終了します。
## 'Latency for Sockshop App' panel を更新する
このサービスレイテンシのグラフは多くの人が閲覧するため、統計的に少なくとも2人は色覚に特性のある方がいることが見込まれます。また、Sockshop のプロダクトオーナーからは色が「つまらない」という指摘がありました。さらに、凡例（legend）も少し整理が必要です。
1. *Latency for Sockshop App* panel を編集します（panel のタイトルにカーソルを合わせ、縦3点メニューをクリックしてコンテキストメニューを表示し、*Edit* をクリック）
2. Visualization Type が *Time Series* になっていることを確認し、なっていなければ切り替えます。
3. まず legend を修正しましょう:
    * 左側の query の Options panel で、Legend type を Verbose から *Custom* に変更し、`{{ job }}` と入力します。グラフの下部に、生の key/value ペアではなく、ジョブ名が表示されるようになります。しかし、namespace の _development_ がまだ表示されているのが気になります。そこで、_transformation_ を使ってフィールド名を変更しましょう。
    * _Transform_ をクリックし、_Rename by Regex_ を選択します（リストをスクロールするか、'Add transformation' の検索を使ってください）。match では、*/* の前後で2つの文字列をキャプチャします。
    * match に `.+/(.+)` と入力します
    * _Replace_ フィールドに `$1` と入力します
4. 秒単位で表示されているこのグラフを、ミリ秒に変換した方が分かりやすいという意見がありました。
    * 左側の query の末尾に *\* 1000* を追加します
    * Y軸に単位を追加します: panel の右上の search options に _Unit_ と入力し、単位として _Time / milliseconds (ms)_ を選択します。
5. 色覚に特性のある同僚にも読みやすくしましょう。
    * グラフ内のすべての線について: *Graph Styles* で Line Width を 2、Fill opacity を 0 に設定します

        ![Graph styles](img/Graph-styles.png)

    * データセット _user_ を破線にします。右上の *Overrides* をクリックし、_Add field override_ を選択します。"Fields with Name" で _user_ を選択します。次に、override property として _Graph styles > Line style_ を追加します。Solid から *Dash* に変更し、*line,space* の設定は *10,10* のままにします。
    * フィールド _payment_ も破線にします。再度 _Add field override_ をクリックし、"Fields with Name" で _payment_ を選択します。次に、override property として _Graph styles > Line style_ を追加します。Solid から *Dash* に変更し、dash の線スタイルは *5,10* を使用します。
    * データセット _catalogue_ も破線にします。Fields with name で _catalogue_ の override を追加し、Line Style の override を追加します。dash パターンは長-短-短にしたいので、*30,3,3*（ドロップダウンの最後の項目）を選択します。
    * Fields with name で _carts_ を選択し、Add override property をクリックします。検索で _Graph styles > Line style_ を見つけます。Solid から *Dots* に変更し、*line,space* の設定は *0,10* のままにします。
    * _orders_ はそのままにしておきます。
6. *Save Dashboard* をクリックします。panel は以下のような表示になるはずです:

    ![Sockshop App](img/sockshop-app.png)

7. もう1つ修正があります！ 2つの青色が似すぎていて区別しにくいので、はっきり区別できるようにしましょう。dashboard 上で、legend にある _user_ に対応する青い線をクリックすると、デフォルトの色一覧が表示されます。Purple を選択してください。
8. dashboard を保存します。
## 'Server Request Rates' panel を time series から bar gauge panel に変換する
最初の panel と同様に、何が良い状態なのかを把握するためのコンテキストが必要です。社内のデータパターンを把握しているため、エンドユーザーのパフォーマンスに影響が出るサービス過負荷の状態を回避したいと考えています。
1. *Server Request Rates* panel を編集します（panel のタイトルにカーソルを合わせ、縦3点メニューをクリックしてコンテキストメニューを表示し、*Edit* をクリック）
2. Panel Title を *Server Request Rates per Second* に変更します（分かりやすくするために "per Second" を追加）
3. Visualization Type を *Time Series* から *Bar Gauge* に切り替えます
4. *Bar Gauge* の設定:
    * Orientation（Layout Orientation）を Auto から Horizontal に変更します。
    * Display Mode を *Gradient* から *Retro LCD* に変更します。
5. Thresholds（右側メニュー panel の下部）の設定:
    * ベースカラーを Green から Blue に変更します
    * 2番目の色を Red から Yellow に変更し、threshold レベルを 80 から 45 に変更します。
    * 3番目の threshold レベル 55 を追加し、色を Orange に設定します。
    * *Save Dashboard* をクリックして panel の設定を適用します。

panel は以下のような表示になるはずです:

![Webserver Request Rates](img/webserver-request-rates.png)

## 既存の library panel を追加する
以前、有用なサービス KPI の panel が Panel Library に保存されていたことを思い出してください。これらを追加することで、サービスの提供状況をより分かりやすく把握できるようになります。

1. dashboard 上で、画面上部の _Add_ ボタンをクリックし、_Import from Library_ を選択します。

2. "Apdex" と検索し、Panel "Service APDEX" を選択します。

    ![Add Library Panel](img/add-panel.png)

    ⚠️ **注意:** この Library Panel はテストデータソースから作成されているため、最初は空白で表示されます。次の手順に進んで、残り2つの panel を追加してください。

2. 同じ手順を繰り返し、"Score" と検索して Panel "Infrastructure - Error Score" を選択します。
3. もう一度同じ手順を繰り返し、"sock" と検索して Panel "Latency Profile, Sockshop Application" を選択します。
5. 重要 - ここまでの作業を保存しましょう！ dashboard を保存してから、リロードして新しい panel を確認してください。

これらの panel を追加すると、上部にリンクアイコンが表示されていることに気づくでしょう。これらの *Panel drilldown* リンクは、より詳細な別の dashboard に遷移します。これで、詳細なサービスビューを一から作る手間が大幅に省けます！

この drilldown 先の dashboard（`Sockshop Performance`）を import するには:

1. 左上のメニューボタン（☰）をクリックし、*Dashboards* をクリックします。
2. Dashboards 画面で *New* ボタンをクリックし、*Import* をクリックします。
2. Import via grafana.com フィールドに `16416` と入力し、*Load* をクリックします。
3. data source を選択するよう求められます:
    - Prometheus (Cloud) には `Prometheus (Cloud)` を選択してください。
4. *Import* をクリックします。

次に、ユーザーが panel のリンクに気づかない場合に備えて、_SLO Status (Errors) per Data Center_ panel（名前を変更した "Error Rates" panel）にも同様の drilldown を追加しましょう。
1. "dull dashboard" を開きます（Home &rarr; Dashboards から見つけてください）。
2. _SLO Status (Errors) per Data Center_ panel を編集し、*Data Links* カテゴリを見つけます（下から3番目です - Panel Links ではなく _Data Links_ です）。
3. _Add Link_ をクリックし、以下を設定します:
    * Title に _Sockshop Service Details_ と入力します。
    * URL に `/d/b2kdXLwnz/sockshop-performance?orgId=1` を貼り付けます。
    * _Open in new tab_ を選択し、*Save* をクリックします。
4. *Save Dashboard* をクリックして Dashboard に戻り、*Save* アイコンをクリックして変更を保存します。
5. 保存が完了したら、SLO Status (Errors) グラフの任意の場所をクリックして、詳細な dashboard に遷移することを確認します。

## 会社のロゴを追加する
見た目にアクセントを加えるため、会社のロゴを追加しましょう:
1. （以前の）地味な dashboard で、_Add_ ボタンをクリックし、_Visualization_ を選択します。
2. 右側でデフォルトの "Time Series" をクリックし、'Text' と検索して *Text* panel を選択します。
3. 右下の _mode_ で、Markdown から HTML に切り替えます。
4. デフォルトのテキストを削除し、以下の HTML を貼り付けます:

    ```html
    <center><img align="center" src="https://i.pinimg.com/originals/74/a0/a5/74a0a51848fb3717c671598dc675c654.jpg" ></center>
    ```

5. Panel Title を削除し、_Transparent Background_ をクリックしてから *Save Dashboard* をクリックします。
6. panel を適切なサイズに調整します。

## panel を配置する
最後に、最も重要なグラフが Z パターンに沿って配置され、適切な間隔とサイズになるように panel を並べ替える必要があります。
dashboard にスペースを追加する必要があるかもしれません。その場合は、library に保存されている空白の text panel を使います。

1. まず、RED metrics（リクエストレート、エラー、レイテンシ）用の row を追加します。
    *  _Add_ ボタンをクリックし、_Row_ をクリックします。
    *  新しい row にカーソルを合わせ、歯車アイコンをクリックして row のタイトルを *Service RED Metrics* に変更します。左から右の順に、Server Requests per Second、SLO Status (Errors) per Data Center、Latency for Sockshop App を上段に移動します。
2. 2番目の row を *Key Performance Indicators* という名前で追加します
    * 残りのグラフをこのグループに移動します。中段は左から右の順に、Service Apdex、Latency quantiles、ロゴを配置します。
    * 下段は、左側に K8s Service Status、その下に Infrastructure - Error Score、その右側に Customer Activity の地図を配置します。
    * row の並び順を変更するには、各 row タイトルの左にある矢印をクリックして row を折りたたみ、row の右端にあるドラッグハンドルを使って上下に移動します。
4. _Add_ をクリックし、_Import from Library_ を選択します。Panel "Blank Space" を選択し、Service RED Metrics の row の後に小さな空白の row を追加します。
5. _Add Panel_ をクリックし、_Import from Library_ を選択します。Panel "Blank Space" を選択し、Key Performance Indicators の上段の後に小さな空白の row を追加します。

panel を配置してスペースを追加すると、dashboard は以下のような表示になるはずです:

![Final-Dashboard One](img/dashboard-one.png)

完了しなかった場合でも、完成版の dashboard を import できます:
import の手順:
1. 左上のメニューボタン（☰）をクリックし、*Dashboards* をクリックします。
2. Dashboards 画面で *New* ボタンをクリックし、*Import* をクリックします。
3. Import via grafana.com フィールドに `16414` と入力し、*Load* をクリックします。
4. data source を3つ選択するよう求められます:
    * TestData DB には `TestData DB` を選択してください。
    * Prometheus (Cloud) には `Prometheus (Cloud)` を選択してください。
    * LokiNginxLogs には `LokiNginxLogs` を選択してください。
    * *Import* をクリックします。
