# きみの かいしゃ しんぶん

7歳の子供が PayPay証券で保有する銘柄について、毎週土曜朝に「今週いくら動いたか」「なぜ動いたか」をひらがなで解説する静的サイト。目的は投資リターンではなく、お金・企業・経済への学習機会をつくること。

- サイト: https://okipoki500.github.io/kaisha-shinbun/
- 銘柄マスタ: `holdings.json`（実際に買った銘柄が変わったらここを書き換えるだけで次号から反映される。**ticker をハードコードしない**）
- 最新号: `index.html`（このファイルを毎週上書きする）
- 過去号: `archive/YYYY-MM-DD.html`（毎週、公開直前の index.html をここに複製してから上書きする。学習の記録として消さない）

## 週次更新エージェントへの指示

### Step 1: 銘柄を読む
`holdings.json` を読み、`stocks` 配列（ticker・日本語名・絵文字・ブランドカラー）を取得する。**このファイル以外に銘柄をハードコードしてはならない**（銘柄が入れ替わった過去の号を模倣して決め打ちしない）。

### Step 2: 株価を取得
各 ticker について直近5営業日の終値を取得する：
```
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/<ticker>?range=5d&interval=1d" -H "User-Agent: Mozilla/5.0"
```
`chart.result[0].timestamp` と `indicators.quote[0].close` から日次終値を組み立て、直近営業日の終値・前日比%・直近5日分の日次騰落（曜日ごとの⤴⤵→）を算出する。

### Step 3: 値動きの理由をWeb調査
各銘柄について WebSearch で直近1週間のニュースを調べ（`<会社名> 株価 ニュース YYYY年M月` のようなクエリ）、値動きの主因を1つ特定する。**情報源が確認できない理由を捏造しない**。材料が特に見当たらない週は「特に大きなニュースはなかったよ。こういう日もあるんだ」で正直に済ませてよい。

### Step 4: 7歳向けに翻訳して執筆
各銘柄カードに以下を書く：
- `なんで？`：値動きの理由を1〜2文、完全ひらがな中心・漢字は最小限（学校で習う範囲を意識）でやさしく説明。抽象的な金融用語（「利益確定売り」等）は使う場合、必ず一言で言い換える
- `おやこトーク`：値動きの理由に関連した問いかけを1文。正解を教える文にしない、考えさせる文にする

### Step 5: サマリーとシミュレーションを計算
- 全銘柄の週間騰落から「一番あがった／さがった」を1行サマリーに
- `holdings.json` の `weekly_amount_yen_per_stock` を各銘柄に投じていた場合の合計額と先週比の増減を計算し「もしも先週、n円ずつ買っていたら」セクションを更新

### Step 6: index.html を再生成
**既存の index.html のデザインシステムをそのまま踏襲する**（新しいレイアウトを考案しない）。変更してよいのは日付・数値・本文コピーのみ。デザイントークン：

- 配色: ライト `--bg:#FBF4E4 --ink:#40372C --up:#E24B2C（あがった＝赤）--down:#2F6FB7（さがった＝青）`、ダークモード対応は `@media (prefers-color-scheme: dark)` と `:root[data-theme]` で既存どおり切り替える
- 各銘柄カードの `--brand` は `holdings.json` の `brand_color` を使う
- フォントはひらがな主体、`.move .word` は「あがった ⤴ / さがった ⤵ / かわらず →」の3値のみ
- `.week` の5マスは月〜金、上昇日は class `u`、下降日は class `d`
- footer に取得日・データ出典（Yahoo!ファイナンス）を明記する

### Step 7: 公開前に前号を退避
```bash
cp index.html "archive/$(TZ=Asia/Tokyo date +%Y-%m-%d).html"
```
その後、新しい index.html で上書きする。

### Step 8: commit & push
```bash
git config user.email "ogiogio321@gmail.com"
git config user.name "okipoki500"
git add index.html archive/ holdings.json
git commit -m "$(TZ=Asia/Tokyo date +%Y-%m-%d) 号"
git push origin main
```

### Step 9: デプロイ確認
push後30〜60秒待ち、`curl -s -o /dev/null -w "%{http_code}\n" https://okipoki500.github.io/kaisha-shinbun/` で `200` を確認する（GitHub Pagesの反映には数分かかることがあるため、404でも即座に異常とはせず1回だけ再確認する）。

## ⚠️ コミット必達フェイルセーフ

どんな理由があっても Step 8（commit & push）に到達するまでセッションを終了しない。ニュースが見つからない・理由がはっきりしない週でも「特に理由は見当たらなかった」で執筆を完了させ、必ず公開する。検索のループに陥ったら即座に打ち切って執筆へ進む。
