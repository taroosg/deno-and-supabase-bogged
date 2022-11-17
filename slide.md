---
transition: "slide"
title: "DenoとSupabaseで沼った話"
---

Deno と Supabase で沼った話

<div style="display:flex;justify-content:space-evenly;align-items:center;background:lightgray;">
<img src="https://seeklogo.com/images/F/fresh-logo-F66F0FD377-seeklogo.com.png" alt="" style="height:120px;">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/84/Deno.svg/1280px-Deno.svg.png" alt="" style="height:150px;">
<img src="https://seeklogo.com/images/S/supabase-logo-DCC676FFE2-seeklogo.com.png" alt="" style="height:130px;">
</div>

---

## 自己紹介

--

|         |                                      |
| ------- | ------------------------------------ |
| Name    | 大杉太郎（たろさん，たろ先生）       |
| Age     | 1987 年生まれ                        |
| Place   | 茨城県 -> 北海道 -> 東京都 -> 福岡県 |
| Career  | 講師，開発（Laravel メイン）         |
| Like1   | Deno，Rust，📚，✈️ 🚃 🚌，🚮         |
| Like2   | 🥃, 🍺, 🍷                           |
| twitter | @taro_osg                            |

--

```txt
                  ＿人人人人人人人人人人人人人人人＿
                  ＞　Deno（Fresh）はいいぞ！！　＜
                  ￣Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y￣
```

---

## もくじ

--

- つくったもの

- 使った技術

- 沼

- まとめ

---

## つくったもの

--

ジーズでは課題を提出する必要がある！

--

提出用のシステムをつくろう！

--

やりたいこと

--

- ブラウザから提出ができる（受講生）

- 提出した内容を確認できる（受講生）

- 提出された内容を評価できる（中の人）

---

## 使った技術

--

(´・ω・｀;)ｳｰﾝ...??

--

(\*`･ω･´) Deno やな！

--

(\*`･ω･´) 最近出た Fresh を使うぞ！

--

(´・ω・｀;) DB は...

--

(\*`･ω･´) Supabase や！

--

(\*`･ω･´) 中身 Postgres やし余裕やろ（慢心）

---

## 沼

--

実装

--

ルーティングつくって（つくってない）

--

ファイルベースでいい感じにやってくれる

```txt
.
├── admin
│   ├── [id].tsx
│   ├── _middleware.ts
│   └── index.tsx
├── index.tsx
└── posts
    ├── [id].tsx
    ├── _middleware.ts
    ├── create.tsx
    ├── fantastic.tsx
    └── index.tsx
```

--

（ディレクトリごとの middleware 設置良き）

--

画面作って．．．

--

SSR のコンポーネントで作成する

```js
import { PageProps } from "$fresh/server.ts";

export default function Greet(props: PageProps) {
  return <div>Hello {props.params.name}</div>;
}
```

--

(\*`･ω･´) とりあえずデプロイするか

--

...

--

(\*`･ω･´) う，動かん．．．

--

---30 分後---

--

(´・ω・｀) モジュールの URL 指定がおかしかった

--

データ入れる処理作って．．．

データ全部もってくる処理つくって．．．

--

(\*`･ω･´) Supabase の公式ライブラリで余裕や！

--

テーブル連携！

```js
const { data, error } = await supabase
  .from("countries")
  .select(`name, cities(name)`);
```

--

（カラム名とか文字列なのが若干キモいな．．）

--

(\*`･ω･´) さて，データの並び替えが必要やな！

--

order()を使う

```js
const { data, error } = await supabase
  .from("cities")
  .select("name", "country_id")
  .order("id", { ascending: false });
```

--

今回は親テーブルの値を

子テーブルのカラムで並び替えたい

--

(\*`･ω･´) fmfm...

```js
const { data, error } = await supabase
  .from("countries")
  .select(`name, cities(name)`)
  .order("name", { foreignTable: "cities", ascending: false });
```

--

...

--

(´・ω・｀;) 順番変わってなくね？？

--

(´・ω・｀;) 修正修正．．．

--

--- 1 時間後 ---

--

(´・ω・｀;;) なんでや？？？

--

(´・ω・｀;;) ん？？

--

```txt
Ordering on foreign tables
doesn't affect the ordering of the parent table.
```

--

```
外部テーブルでの順序付けは、
親テーブルの順序付けに影響しない。（DeepLによる翻訳）
```

--

(´・ω・｀;;) あああああああああああ

--

並び替わらないのは「仕様」

--

```txt
              ＿人人人人人人人人人人人人人人＿
              ＞　ドキュメントを読め！！！　＜
              ￣Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y￣
```

--

(\*`･ω･´) クソがあああああああ！！！

--

力こそパワー

```js
posts.sort((a, b) => {
  if (a.class > b.class) return -1;
  if (a.class < b.class) return 1;
  if (a.work > b.work) return -1;
  if (a.work < b.work) return 1;
  if (a.student_number > b.student_number) return 1;
  if (a.student_number < b.student_number) return -1;
  return 0;
});
```

--

![やれやれだぜ](https://stat.ameba.jp/user_images/20201006/22/funnymanbuggy/4c/e7/j/o0378054714830956365.jpg)

---

## まとめ

--

Deno（Fresh）はいいぞ！

--

Supabase は

外部テーブルキーだと並び替わらないよ！

--

JS でなんとかすればなんとかなるよ！

--

```txt
              ＿人人人人人人人人人人人人人人＿
              ＞　ドキュメントを読め！！！　＜
              ￣Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y￣
```
