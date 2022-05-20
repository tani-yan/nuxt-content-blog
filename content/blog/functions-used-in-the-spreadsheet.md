---
title: スプレッドシートで使った関数など
description: Generate blog from Markdown with nuxt/content
createdAt: 2021-10-14
tags:
  - スプレッドシート
  - GAS
---

## Googleフォームが送信されたら回答にIDをつける
1. フォームと連携させたスプレッドシートでIDのセルを作成。
1. [**ツール**] > [**スクリプトエディター**] でエディターを開き、下記スクリプトを貼り付ける。
``` js
function setId() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sht = ss.getSheetByName("シート名");
  var range = sht.getRange(sht.getLastRow(),1);
  if(range.isBlank() == true){
    range.setValue("=ROW()-1");
  }
}
```

3. コードを保存し、[**トリガー**] > [**トリガーを追加**] > [**イベントの種類を選択**] >[**フォーム送信時**] > [**保存**]
3. エラーが出て失敗したらもう一度保存をクリックし、 [**Googleアカウントを選択**] > 左下の [**詳細**] > [**プロジェクトに移動**] > [**許可**]


## 他のスプレッドシートのデータをインポート
```zsh
=ARRAYFORMULA(IMPORTRANGE("シートURL","'シート名!'!開始範囲:終了範囲"))
```

※セルが#REF!となって表示されなかったらクリックしてアクセスを許可。
