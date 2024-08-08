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


## Ch5: Navigating Between Pages
- `Link`を使うと、ページ丸ごとリロードすることなく遷移できる
  - 変わってないじゃん？と思ったが、ブラウザのページのアイコンがぐるぐるしているかどうかで判定できる
- コードを**prefetch**してくれるので、クリック時にはすでに目的のページが裏でロードされてる
  - めちゃはやく表示できる
  - 参考：https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works
- 