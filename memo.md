# Ch3: Optimizing Fonts and Images

## Image

- `md:block` <-これは、`mobile`?`middle device`
- [Image Optimization](https://nextjs.org/docs/app/building-your-application/optimizing/images)

# Ch4: Layouts and Pages

## Nested Routing

- フォルダ配下の page.tsx は特別で、例えば app/page.tsx は /, app/oyo/page.tsx は /oyo にアクセスすると表示される
  <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Ffolders-to-url-segments.png&w=3840&q=75">

## layout

- `layout.tsx`は他のページと共有の UI を作成できる
  <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fshared-layout.png&w=3840&q=75">
- page コンポーネントは再レンダリングされるけど、レイアウトはされないので、（多分効率が良い）

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

- データを DB に登録することを`seeding`と呼ぶ？

# [Ch.7: Fetching Data](https://nextjs.org/learn/dashboard-app/fetching-data#using-server-components-to-fetch-data)

## Choosing how to fetch data

API

- 外部の API を提供してるサービスを使いたい時
- データを秘密にしておきたい時
- https://nextjs.org/docs/app/building-your-application/routing/route-handlers

データベースクエリ

- DB をいじいじする必要があるとき
- React Server Component を使ってるとき
  - なんだこれは？
  - DB を直接クエリ操作できる。DB の漏洩をリスクがない
- https://vercel.com/docs/storage/vercel-postgres/using-an-orm

SQL

- なんでもできる（versatile: 用途が広い）

結局、API を使っても、クエリを使っても、SQL を使うんじゃないのか？

## React Server Components

どうやら新しい機能？らしい。デフォで使われてる

メリット

- 非同期処理が楽になる？`useEffect`/`useState`を使わなくても、`async/await`が使える？
- サーバーで実行されるから
  - 結果のみクライアントに送られる（データ・ロジックはサーバーのみが保持できる）
  - 直接 DB をクエリ操作できる

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
  - SQL が問い合わせてる先のデータが削除・追加されても、そのつど SQL が実行されるわけじゃない
-

## fetching data for the dashboard overview page

- `export default async function Page()`の`async`が`await`を使えるようにする
- `` const data = await sql<Revenue>`SELECT * FROM revenue`; ``で SQL が実行できそう

# Ch.8: Static and Dynamic Rendering

- 静的レンダリング：ユーザーが訪れるたびに、キャッシュされたのが提供される
- メリット
  - 早い
  - サーバーが大変じゃない
  - SEO 対策になる。クローラーがニコニコする
- 向いてる場所
  - データ使わないサイト
  - みんな共通のデータ（不変）を使う
- 動的レンダリング
  - クッキーとか、URL のパラメータとかの、リクエスト時にのみ得られる情報にアクセスできる
  - ただ、データのフェッチに時間がかかると、ページ読み込みもその分遅くなる

# Ch.9: Streaming

## Stream について学ぼう

- データを小さいまとまり（チャンク）にして、送れるぜ！になったら各々送る
  - 早くなるよ
    <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fserver-rendering-with-streaming.png&w=1920&q=75">
    <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fserver-rendering-with-streaming-chart.png&w=1920&q=75">
- 方法

1. loading.tsx ファイル

- `loading.tsx`は next.js の`Suspense`の特別なファイルらしい
  - `page.tsx`みたいな、特殊な奴か？

2. `<Suspense>`コンポーネント

- 何らかの条件が満たされるまで（例：データが読み込まれるまで）一部のレンダリングを延期できる
- 動的コンポーネントを Susppense でラップ、動的コンポーネントのロード中に表示するフォールバックコンポーネントを渡す

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
     - クライアントの JS が頑張らなくていい
     - データがクライアントに漏れる心配ない
  3. 必要な分のデータは SQL で取得しよう
     - 毎回の通信するデータが減る
     - in-memory に変換するようの JS が減る
  4. データは並列に取得しよう
  5. Streaming で、重いデータが足を引っ張ってページがいつまでも表示されない...をなくそう
     - すべてロードされなくても、ユーザーが画面操作できるようにしたいね
  6. データフェッチを必要なコンポーネントに移すことで、ルートのどの部分をダイナミックにすべきかを分離します。（？？？）
     - ちょっと何言ってるか分からない...?????

# [Ch.11: Adding Search and Pagination](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination)

- Pagination?: 検索件数がたくさんあるときに、下に出てくるあれ
  <img src = "https://s3-alpha.figma.com/hub/file/3328835529/e6b30bf5-dcdd-4430-a971-97706bd0fe05-cover.png">

## なんでURL search paramを使う？
- 他の方法として、クライアントの方でstate管理もできる
- メリット
  1. ブックマークや、他の人にURLでシェアできる
  2. サーバーサイド・レンダリングと初期ロード：初期状態をレンダリングするために、URLパラメータをサーバー上で直接消費することができ、サーバーレンダリングの処理が容易になります。？？？
     1. 良く分からない？？？
  3. 分析・トラッキング：クライアントでごちゃごちゃしなくても、クエリ・フィルターがURLに直接埋め込まれてるので、容易にユーザーの挙動を追跡できる

## 検索機能で使う機能
- `useSearchParams`: URLのパラメータを取得可能
  - `/dashboard/invoices?page=1&query=pending` -> `{page: '1', query: 'pending'}.`
- `usePathName`: 現在のURLを取得可能
- `useRouter`: ページの遷移が出来る（？）

## debouncing（ディバウンシング）
- 関数の発火を制限する
  - 現状：ユーザーが一文字入力するたびに、検索している->サーバーが大変
  - 理想：ユーザーが入力を止めた時に、発火させたい
- 原理
  1. 発火：発火したら、タイマーがスタート
  2. 待機：もし再度発火 & タイマーが時間切れじゃない -> タイマーリセット
  3. 実行：もしタイマーが切れたら、関数実行
`pnpm i use-debounce`


## Tip: defaultValue vs value
- `value`: stateで入力値を管理してる
  - controlled components(?)にするため
  - reactがinputの状態を管理できる
- `defaultValue`: stateで管理してない
  - inputのみが情報を管理・保持してる
```
<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get('query')?.toString()}
/>
```

## `useSearchParams` hook vs `searchParams` props
- `useSearchParams`:  クライアントコンポーネント
- `searchParams`: サーバーコンポーネント

# Ch.12: Mutating Data (mutate: 変化する)
## Server Actionとは
- サーバー上でコードを非同期で実行できる
  - APIを作らなくてもよい！
  - クライアント/サーバーコンポーネントから実行可能
- どうやら、セキュリティ面もいい感じらしい
  - encrypted closures, strict input checks, error message hashing, and host restrictions...

## invoiceを作ろう
- `use server;`: エクスポートされたのが、server actionになる。
  - クライアント・サーバーコンポーネントにて使用可能
- `formData`の情報を取得するためには、`.get(name)`を使う
  - 
  ```
  'use server';
 
  export async function createInvoice(formData: FormData) {
    const rawFormData = {
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    };
  }
  ```
- バリデーションしたい
  - `typeof rawFormData.amount`: 型が分かる
  - [Zod](https://zod.dev/): バリデーションライブラリ
  - 
  ```
  import { z } from 'zod';
  const rawFormData = {
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  };
  ```
  - floatをなるべく扱わないようにしよう(intで管理できるのであれば、そうする)
- [client-side router cashe](https://nextjs.org/docs/app/building-your-application/caching#router-cache)
  - これ、良く分からない

## invoiceをアップデートしよう
- [Dynamic Route Seements](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes)
  - 正確な「segment name」を知らない & ルートをデータを基に作りたい時に良い (ブログタイトル、製品ページ...)
  - <img src="https://nextjs.org/_next/image?url=%2Flearn%2Flight%2Fedit-invoice-route.png&w=3840&q=75">
- server actionにはpropsを渡せない？bindを使う必要がある？
  - これ良く分からない

## Tip: UUID vs auto-incrementing key
- auto-incrementing key: 短い
- UUID: 長いけどたくさんのメリット
  - IDが衝突しない・ユニーク・列挙型攻撃に強い -> 大きいDBに向いてる

# Ch.13: Handling Errors
[error.js](https://nextjs.org/docs/app/api-reference/file-conventions/error)
[notFound](https://nextjs.org/docs/app/api-reference/functions/not-found)
[not-found.js](https://nextjs.org/docs/app/api-reference/file-conventions/not-found)

## error.tsx
- エラーが発生したら表示するページ？
  - どんなエラーでも！
- fallback(前の画面に戻る？)UIを表示する
- クライアントコンポーネント（`use client`）
- 2つのpropsを受け取る
  - `error`: JSのError オブジェクト
  - `reset`：route segment(前の画面？)を再レンダリングする
- ファイルを作って、中身を書く
## `notFound`関数
- リソースが存在しない時のみ
- 下のように、エラーが出そうなところでハンドリング、同じ階層に`not-found.tsx`を作成
  ```tsx
  import { notFound } from 'next/navigation';
  if (!invoice) {
        notFound();
  }
  ```

# [Ch.14: Improving Accessibility](https://nextjs.org/learn/dashboard-app/improving-accessibility)
## Form Validation
- `useActionState`(クライアントコンポーネントで使う)
```tsx
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: 'Please select a customer.',
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: 'Please enter an amount greater than $0.' }),
  status: z.enum(['pending', 'paid'], {
    invalid_type_error: 'Please select an invoice status.',
  }),
  date: z.string(),
});
```
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

`import { lusitana } from '@/app/ui/fonts';`と `import CardWrapper from '@/app/ui/dashboard/cards';`は何が違う？
なんで二通りの書き方が存在する？

## `use client`?
- イベントリスナーとフックを使えるらしい