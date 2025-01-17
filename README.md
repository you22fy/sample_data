# sample_data
[NTCIR-18 U4タスク](https://sites.google.com/view/ntcir18-u4/home, "NTCIR-18 U4")（**U**nifying, **U**nderstanding, and **U**tilizing **U**nstructured Data in Financial Reports
）のサンプルデータです。\
U4タスクは、 **Table Retrievalサブタスク（以下、TRタスク）** と、 **Table QAサブタスク（以下、TQAタスク）** から構成されます。

## 更新情報
- (2024/5/30) sampleデータの公開

## タスク設定
以下をご参照ください。\
[TRタスク](https://sites.google.com/view/ntcir18-u4/subtasks/table-retrieval, "Table Retrieval")\
[TQAタスク](https://sites.google.com/view/ntcir18-u4/subtasks/table-qa, "Table QA")

## 配布ファイル
このリポジトリには以下のファイルが含まれます。

- `train/question_information.json`
    - 各タスクのトレーニングに必要となる情報を記したもので、質問文に対する各タスクのコンテキストと解答が記載されています。
- `train/reports/*.html`
    - 各企業が発行した有価証券報告書に情報を追加したもので、各タスク共通の train データとして使用します。
- `test/answersheet.json`
    - 質問文とコンテキストが記載されており、それに対する解答が空欄であるJSONファイルです。解答を埋める形式で、各タスク共通の test データとして使用します。
- `test/reports/*.html`
    - 各企業が発行した有価証券報告書に情報を追加したもので、各タスク共通の test データとして使用します。
- `test/gold.json`
    - `test/answersheet.json`の解答を埋めたもので、評価のために使用します。
<!-- - `src/baseline.py`
    - サンプルの推論スクリプトです。ランダムに解答を埋めます。 -->
- `src/eval.py`
    - 評価スクリプトです。TRタスク、TQAタスク、トータルの3項目について、Accuracy と、F1-score を出力します。TRタスクのみ、あるいはTQAタスクのみ回答の場合でも、評価は可能です。
    - 引数として、`-g [Goldデータ.jsonのパス] -i [答えを埋めたanswersheet.jsonのパス]` を取ります。
<!-- - `src/convert.py`
    - train データの形式から、提出用のファイル形式に変換します。
    - train データの一部を dev データとして使用する場合の、評価スクリプトに与える gold データを作成するために使用してください。 -->

## 入力ファイル形式
### HTMLファイル
各企業が発行した有価証券報告書（HTML 形式）に、必要なアノテーションを行ったものを利用します。

`reports/*.html` は、各企業が発行した有価証券報告書に、以下の修正を加えたものです。

- `table` タグに `table-id` 属性を追加。
    - `table-id`は、テーブルを一意に識別する文字列で、`[書類管理番号(DocID)]-[ファイル名]-tab[テーブル連番]`の形式です。
    - 例：`S100IHTB-0000000-tab1`
- `th` タグ及び `td` タグに `cell-id` 属性を追加。
    - `table-id`は、テーブルを一意に識別する文字列で、`[書類管理番号(DocID)]-[ファイル名]-tab[テーブル連番]-r[列番号]c[行番号]`の形式です。
    - 例：`S100IHTB-0000000-tab1-r3c2`

### JSONファイル
`test/answersheet.json` は、質問文に対する各タスクのコンテキスト（検索範囲）が記載されており、解答は空欄です。`train/datainfo.json`は、トレーニングデータとしての使用を想定し、正しい解答を埋めたものです。

```json
{
    "QuestionID": {
        "question": "質問文",
        "doc_id": "DocID",
        "table_id": "TableID",
        "cell_id": "CellID",
        "value": "セルの値"
    },
}
```

各パラメータの詳細を以下に示します。

| 要素名 | 型 | 説明 | 例 |
| --- | --- | --- | --- |
| question | string | 質問文。TRタスク、TQAタスク共にqueryとして用いる。 | 大和ハウス工業の2019年の個別のShareholdersEquityにおける「自己株式の処分」を含む表は？ |
| doc_id | string | 検索対象のHTMLが格納されたディレクトリ名。 <br> TRタスクにおいて、contextとして用いる。 | S100ITAZ |
| table_id | string | 任意の`table`タグに付与された、`table-id`属性。 <br> TRタスクではanswer、TQAタスクではcontextとして用いる。 | S100ITAZ-2020-tab1 |
| cell_id | string | 任意の`tr`あるいは`td`タグに付与された、`cell-id`属性。 <br> TQAタスクにおいて、answerの一つとして用いる。 | S100ITAZ-2020-tab1-r3c2 |
| value | string | 任意のセルの値。金額表現の場合は、桁数を考慮したものとなる。 <br> TQAタスクにおいて、answerの一つとして用いる。 | 3812000000 |

## 出力ファイル形式
いずれのタスクも、`test/answersheet.json`の解答部分を埋める形式で、出力ファイルを作成してください。
- task1の`answer`には、`S100ITAZ-0101010-tab1`というように、予測したTableIDを入力してください。
- task2には、`answer_id`と`value`の2種類の回答箇所が存在します。いずれか片方のみに予測結果を入力してください。
    - `answer_id`には、`S100ITAZ-0101010-tab1-r3c2`というように、予測したCellIDを入力してください。
    - `value`には、`3812000000`というように、予測したデータそのものを入力してください。

## 出典

- `train` ならびに `test` ディレクトリ内のファイルは、EDINET 閲覧（提出）サイト（※）をもとに NTCIR-18 U4 タスクオーガナイザが作成したものです。
    - （※）例えば書類管理番号が `S100ISN0` の場合、当該ページの URL は `https://disclosure2.edinet-fsa.go.jp/WZEK0040.aspx?S100ISN0` となります。書類管理番号は、`train`/`test` ディレクトリ内の各ファイル名の先頭 8 文字です。

- 東証33業種分類の大分類（全10種）をベースに、sample dataの対象となる有報を決定しました。\
training dataは、TOPIX100の算出対象企業のうち、3社以上が含まれる6業種から各1社、計6社の有報を対象としました。

    | 大分類 | 対象企業 | DocID |
    | --- | --- | --- |
    | 水産・農林業 | 該当なし | None |
    | 鉱業 | 該当なし | None |
    | 建設業 | 該当なし | None |
    | 製造業 | 株式会社バンダイナムコホールディングス | S100ISF1 |
    | 電気・ガス業 | 該当なし | None |
    | 運輸・情報通信業 | Zホールディングス株式会社 | S100IT7Z |
    | 商業 | 伊藤忠商事株式会社 | S100ITS8 |
    | 金融・保険業 | オリックス株式会社 | S100J36O |
    | 不動産業 | 三井不動産株式会社 | S100IU3A |
    | サービス業 | エムスリー株式会社 | S100J50B |
