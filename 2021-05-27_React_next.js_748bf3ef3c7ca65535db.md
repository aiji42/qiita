<!--
title:   【Next.js】Warning: X did not match.Server: y Client: z は絶対に解消しろ
tags:    React,next.js
id:      748bf3ef3c7ca65535db
private: false
-->
## 結論
パフォーマンスに与える影響が非常に大きい。
「コンソールにワーニング出てるけど、レンダリングされる結果は問題ないからいいか」ぐらいの軽い気持ちで無視すると痛い目を見る



|  | Warning解消前 | Warning解消後 |
| ---- | ---- | ---- |
| 1回目 | ![mismatch.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/46467/c237d465-0355-45e8-6c6d-3d826c484187.png) | ![match.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/46467/1c6aa532-4bcb-f615-3395-4969d749dce5.png) |
| 2回目 | ![スクリーンショット 2021-05-27 10.48.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/46467/40531753-d560-854e-7652-64b4254c65c0.png) | ![スクリーンショット 2021-05-27 10.49.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/46467/4eb199a8-4cbf-3a48-bfd1-b1fe62ab07b6.png) |
ちなみに、このサンプルはJSON-LD(構造化マークアップデータ)の出力がサーバサイドとクライアントサイドに微妙な差異が生じていたときの例。コンテンツ量が多めのページで比較すると、差が顕著に出る。

ちなみに、パフォーマンスプロファイルで見ると、「before-hydrate」 + 「hydrating」 に100~300msの差が出ていた

SSG/SSR しているのに、CSRしているのと何も変わらない状態になる。

## 何が起こっているのか

### ReactDOMServer と Hydrate

CSR な React アプリケーションは `ReactDOM.render` を使って DOM をマウントしている。
React でアプリケーションを構築した人は、必ずと言ってよいほど見たことがあるだろう。

```js
ReactDOM.render(App, document.getElementById('root'))
```

Next.js や Gatsby などの、 SSR/SSG を行うフローではこのマウントの方法が少し異なる。

![引用: How to implement server-side rendering (SSR) in your React application with NodeJS – step by step tutorial
](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/46467/e88ca5f2-8be0-7673-4226-f7c0c72d80d7.png)

https://lebersoftware.hu/react-server-side-rendering-step-by-step-tutorial/

上の画像の2番では、`ReactDOMServer` によって、React のコードから HTML が生成される。  
https://ja.reactjs.org/docs/react-dom-server.html
(このときの HTML は、`useEffect` のような、マウントに対しての副作用的な処理が実行される前の HTML であることをイメージとして持っておくと良い。「マウント」はクライアント側でしか発生しない。)

その後、HTML と JS がクライアントに送信され、5番が処理される。
このとき、`ReactDOM.render` ではなく `ReactDOM.hydrate` が実行される。
https://ja.reactjs.org/docs/react-dom.html#hydrate

```js
ReactDOM.hydrate(App, document.getElementById('root'))
```

`ReactDOM.hydrate` は、 サーバサイドで生成された HTMLのDOM構造と、クライアントサイドで生成された仮想DOMが一致していることを期待している。
一致していれば、DOMの再レンダリングをスキップし、イベントリスナーの登録だけを行う。
もし一致しなければ、(可能な限り)DOMの状態を再現するために再レンダリングを行う。
> React はレンダーされる内容が、サーバ・クライアント間で同一であることを期待します。React はテキストコンテンツの差異を修復することは可能ですが、その不一致はバグとして扱い、修正すべきです。開発用モードでは、React は両者のレンダーの不一致について警告します。不一致がある場合に属性の差異が修復されるという保証はありません。これはパフォーマンス上の理由から重要です。なぜなら、ほとんどのアプリケーションにおいて不一致が発生するということは稀であり、全てのマークアップを検証することは許容不可能なほど高コストになるためです。

### つまり

サーバサイドで生成しテキスト化したDOMと、クライアントサイドで生成(計算)したDOMとの間に、1文字でも差分が生じると、ページ全体が再レンダリングされることになる。
特定のコンポネント一箇所だけで起こっていたとしても、`ReactDOM.hydrate`はルートエレメントに対して行われるため、対象コンポネントだけ再レンダリングというわけにはいかない。この処理がかなり高コストであり、非力なモバイル端末だとパフォーマンスに影響がでる。

## 対処

対処方法に関しては、下の記事を参考にされたし。
https://zenn.dev/takewell/articles/5ee9530eedbeb82e4de7

先に述べた通り、`useEffect` は Hydrate 後に行われるため、`useEffect` と `useState` を駆使して、マウントされているかどうかで分岐すると問題は最小限に留められる。
(「最小限に留められる」というのは、あくまでレンダリングを遅延させただけで、そのコンポネントに関してはSSG/SSRの恩恵を受けられなくなるため。)