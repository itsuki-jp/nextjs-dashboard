# Ch3: Optimizing Fonts and Images
## Image
- `md:block` <-これは、`mobile`?`middle device`
- https://nextjs.org/docs/app/building-your-application/optimizing/images

# Ch4: Layouts and Pages
## Nested Routing
- フォルダ配下のpage.tsxは特別で、例えば app/page.tsx は /, app/oyo/page.tsx は /oyo にアクセスすると表示される
<img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Ffolders-to-url-segments.png&w=3840&q=75">

## layout
- `layout.tsx`は他のページと共有のUIを作成できる
<img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fshared-layout.png&w=3840&q=75">
- pageコンポーネントは再レンダリングされるけど、レイアウトはされないので、（多分効率が良い）


# Ch5: Navigating Between Pages
## Link
- `Link`を使うと、ページ丸ごとリロードすることなく遷移できる
  - 変わってないじゃん？と思ったが、ブラウザのページのアイコンがぐるぐるしているかどうかで判定できる
- コードを**prefetch**してくれるので、クリック時にはすでに目的のページが裏でロードされてる
  - めちゃはやく表示できる
  - 参考：https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works

## usePathname()
- クライアントコンポーネント？にする必要がある？
- 参考：https://nextjs.org/docs/app/api-reference/functions/use-pathname

## clsx
```
className={clsx(
    'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
    {
    'bg-sky-100 text-blue-600': pathname === link.href,
    },
)}
```
- `pathname === link.href`の時に、下部を適用する、って認識であってるかな


# Ch.6: Setting Up Your Database
- データをDBに登録することを`seeding`と呼ぶ？

# [Ch.7: Fetching Data](https://nextjs.org/learn/dashboard-app/fetching-data#using-server-components-to-fetch-data)
## Choosing how to fetch data
API
- 外部のAPIを提供してるサービスを使いたい時
- データを秘密にしておきたい時
- https://nextjs.org/docs/app/building-your-application/routing/route-handlers

データベースクエリ
- DBをいじいじする必要があるとき
- React Server Componentを使ってるとき
  - なんだこれは？
  - DBを直接クエリ操作できる。DBの漏洩をリスクがない
- https://vercel.com/docs/storage/vercel-postgres/using-an-orm

SQL
- なんでもできる（versatile: 用途が広い）

結局、APIを使っても、クエリを使っても、SQLを使うんじゃないのか？
## React Server Components
どうやら新しい機能？らしい。デフォで使われてる

メリット
- 非同期処理が楽になる？`useEffect`/`useState`を使わなくても、`async/await`が使える？
- サーバーで実行されるから
  - 結果のみクライアントに送られる（データ・ロジックはサーバーのみが保持できる）
  - 直接DBをクエリ操作できる

問題点 
- `request waterfall`が発生する
  - 前のリクエストが終了したら、次のが実行される。並列処理されてない
    ```
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices(); // fetchRevenue()が終わるまで待つ
    const {
    numberOfInvoices,
    numberOfCustomers,
    totalPaidInvoices,
    totalPendingInvoices,
    } = await fetchCardData(); // fetchLatestInvoices()が終わるまで待つ
    ```

    - 対処法として、`Promise.all()`で並列に処理
      - どれかのリクエストがめっちゃ遅かったらどうなる...?(そんなこと聞かれても、知らない)
      - これ、並列に実行している場合ではなく、非同期処理してる場合は、一つ遅いのがあると大変だよね！っていう話な気がしてきた
        - 非同期だろうが、同期だろうが、データを動的に読み込むのであれば、遅い処理があればそりゃページ読み込む速度遅くなるよね

    ```
    export async function fetchCardData() {
    try {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
            SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
            SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
            FROM invoices`;
    // Promise.all()で並列に処理
    + const data = await Promise.all([
        invoiceCountPromise,
        customerCountPromise,
        invoiceStatusPromise,
    ]);
    // ...
    }
    }
    ```
  - <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fsequential-parallel-data-fetching.png&w=3840&q=75">
- `static rendering`(静的レンダリング)なので、データが変化しても表示は変化しない
  - SQLが問い合わせてる先のデータが削除・追加されても、そのつどSQLが実行されるわけじゃない
- 

## fetching data for the dashboard overview page
- `export default async function Page()`の`async`が`await`を使えるようにする
- ``const data = await sql<Revenue>`SELECT * FROM revenue`;``でSQLが実行できそう

# Ch.8: Static and Dynamic Rendering
- 静的レンダリング：ユーザーが訪れるたびに、キャッシュされたのが提供される
- メリット
  - 早い
  - サーバーが大変じゃない
  - SEO対策になる。クローラーがニコニコする
- 向いてる場所
  - データ使わないサイト
  - みんな共通のデータ（不変）を使う
- 動的レンダリング
  - クッキーとか、URLのパラメータとかの、リクエスト時にのみ得られる情報にアクセスできる
  - ただ、データのフェッチに時間がかかると、ページ読み込みもその分遅くなる

# Ch.9: Streaming
## Streamについて学ぼう
- データを小さいまとまり（チャンク）にして、送れるぜ！になったら各々送る
  - 早くなるよ
  <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fserver-rendering-with-streaming.png&w=1920&q=75">
  <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fserver-rendering-with-streaming-chart.png&w=1920&q=75">
- 方法
1. loading.tsxファイル
- `loading.tsx`はnext.jsの`Suspense`の特別なファイルらしい
  - `page.tsx`みたいな、特殊な奴か？
2. `<Suspense>`コンポーネント
- 何らかの条件が満たされるまで（例：データが読み込まれるまで）一部のレンダリングを延期できる
- 動的コンポーネントをSusppenseでラップ、動的コンポーネントのロード中に表示するフォールバックコンポーネントを渡す

# [Ch.10: Partial Prerendering](https://nextjs.org/learn/dashboard-app/partial-prerendering)
- 元々、ルート（？）全体を静的・動的レンダリングのどちらかに統一する必要があった（？）
  - `if you call a dynamic function in a route (like querying your database), the entire route becomes dynamic.`
  - でも、動的・静的どっちも使いたいね
  - <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fdashboard-static-dynamic-components.png&w=3840&q=75">

- Partial Prerendering(PPR): Next.js 14 で登場
  - `Suspense`を使って、動的に読み込む場所のスペースを残して静的レンダリング->動的コンテンツを非同期に読み込みレンダリング

- 使い方
  - `Suspense`で動的なのをラップしてれば、基本的にコードを変えなくてもよい
```neext.config.mjs
// 追記
const nextConfig = {
    experimental: {
        ppr: "incremental",
    }
};
```

```layout.tsx
export const experimental_ppr = true; // 追記
```

- データフェッチを最適化しよう
  1. データベースのリージョンは近所に
  2. `React Server Components`を使うと、ロジック・データがサーバー内のみに存在する
     - クライアントのJSが頑張らなくていい
     - データがクライアントに漏れる心配ない
  3. 必要な分のデータはSQLで取得しよう
     - 毎回の通信するデータが減る
     - in-memoryに変換するようのJSが減る
  4. データは並列に取得しよう
  5. Streamingで、重いデータが足を引っ張ってページがいつまでも表示されない...をなくそう
     - すべてロードされなくても、ユーザーが画面操作できるようにしたいね
  6. データフェッチを必要なコンポーネントに移すことで、ルートのどの部分をダイナミックにすべきかを分離します。（？？？）
     - ちょっと何言ってるか分からない...?????


# memo
## よくわからんエラー
```
Next.js (15.0.0-canary.56) is outdated (learn more)

Unhandled Runtime Error
Error: Unsupported Server Component type: undefined
```
- これは`pnpm add next@canary`したら直った。
- `npm i next@latest`, `npm i next@canary`は無力だった

## import の謎
`import { lusitana } from '@/app/ui/fonts';`と  `import CardWrapper from '@/app/ui/dashboard/cards';`は何が違う？
なんで二通りの書き方が存在する？