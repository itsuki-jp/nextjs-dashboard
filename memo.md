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

# memo
## よくわからんエラー
```
Next.js (15.0.0-canary.56) is outdated (learn more)

Unhandled Runtime Error
Error: Unsupported Server Component type: undefined
```
- これは`pnpm add next@canary`したら直った。
- `npm i next@latest`, `npm i next@canary`は無力だった