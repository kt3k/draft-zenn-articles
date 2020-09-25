---
title: "Capsid.js について"
emoji: "💊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frontend", "javascript", "typescript", "dom"]
published: false
---

Capsid.js は一言でいうと**普通の DOM Programming**をするためのツールです。

普通の DOM Programming って何という話ですが、ここでは以下のようなものを普通の DOM Programming と考えることにします。

- UI の挙動は DOM Event に対するハンドラーとして記述する
- UI の挙動は HTML の class 単位でまとめる (この単位をコンポーネントととらえる)
- コンポーネントの挙動は、コンポーネント内には影響を及ぼすが、コンポーネント外には影響しない (local な作用を意識する)
- コンポーネント間の連絡は基本的に DOM Event を介して行う

大雑把にまとめると、ローカルな作用を意識したコンポーネント達が DOM Event を投げ合うことで連絡しあって、UI を成すという考え方です。

## ローカルな作用

ローカルな作用についてもう少し詳しく説明すると、あるコンポーネントにとって、ローカルな作用とは、そのコンポーネントのルートとなる DOM 要素からみて子孫となる DOM 要素に対する編集作業の事をローカルな作用と考えます。子要素、孫要素に対する操作はローカルな作用、兄弟要素や親要素、さらに遠い親戚要素に対する操作はローカルではない作用と考えます。

なぜローカルな作用に限定する必要があるのでしょうか?

大規模な jQuery プロジェクトの開発体験を思い出してみましょう。大規模な jQuery コードベースはうまくいかないというのが

## 例

このような考えに沿ったコンポーネントの実装は例えば以下のようになります。

```js
// edit-modal コンポーネントの実装
// このコンポーネント実装の中では el (.edit-modal 1個) の中にしか触らないようにする
// 外部に通信する必要がある場合はイベントを発行して bubbling でキャッチ等させる
document.querySelectorAll('.edit-modal').forEach(el => {
  // custom event の `open-modal` を受け取ったらモーダルを開く
  el.addEventListener('open-modal', () => {
    el.classList.add('show')
  })

  // ok-btn がクリックされたら API を呼び出す
  el.querySelector('.ok-btn')?.addEventListner('click', async () => {
    try {
      await callApi()
    } catch(e) {
      showError(e)
    }
    el.dispatchEvent(new CustomEvent('editted'))
  })

  ...
})
```

このようなコードを書くことで、コンポーネントの実装はコンポーネントの中で閉じた状態になり、外部への依存が無い状態になります。

このような状態にすることのメリットの一つは、コードが読みやすいということです。このコンポーネントの挙動はこの実装が書かれたファイル内で閉じるため、このファイルだけを読めばこのコンポーネントを理解できます。

また、他のメリットはこのコンポーネントがテストしやすいということです。DOM Event の入出力を検証できれば、すべての振る舞いを検証できるため、JSDOM のような DOM 実装のある環境であれば、このコンポーネントのユニットテストを、それほどの障害なく書くことが出来ます。ただし、API のリクエストなど環境に作用する処理はモックなどが必要になりますが。

```ts
// 適切に初期化した上で
it('open-modal イベントを受け取って .show class を付与する', () => {
  const el = document.querySelector('.edit-modal')

  el.dispatchEvent(new CustomEvent('open-modal'))

  expect(el.classList.has('show')).toBe(true)
})
```

また、テストしやすいことはすなわちドキュメントしやすいということも意味します。ローカル作用を意識したコンポーネントは入出力が絞られるためドキュメントも書きやすくなります。

このように、ローカルな作用を意識したイベント駆動プログラミングでは非常に高い生産性を実現することが出来ます。

# Capsid.js

Capsid.js は上のような、ローカルな作用を意識したイベント駆動プログラミング (locality-aware event-driven programming) を支援するためのツールです。上の edit-modal の例は capsid.js では以下のように書くことが出来ます。

```js
@component('edit-modal')
class EditModal {
  @on('open-modal')
  onOpen() {
    this.el.classList.add('show')
  }

  @on('click', { at: '.ok-btn' })
  @emits('editted')
  async onEdit() {
    try {
      await callApi()
    } catch(e) {
      showError(e)
    }
  }
}
```

`@component` のような `@` ではじまる記法は JavaScript のデコレータです。このデコレータは TypeScript の experimental デコレータとも互換性があり、TypeScript のトランスパイラーを使って capsid.js を正しく使うことが出来ます。

`@component('.edit-modal')` を class に付加することで、このコンポーネントの挙動が `.edit-modal` クラスに自然に結びつきます。`on('...')` はイベントハンドラーを登録するデコレータです。このデコレータを付けることで、そのメソッドが .edit-modal クラスの DOM に登録されたイベントハンドラーになります。jQuery の on 関数には、非常に便利な event の delegate という機能がありました。`@on('click', { at: '.ok-btn' })` というデコレータはそのようなイベントのデレゲートを表現しています。このデコレータがついたメソッドはコンポーネントの DOM のイベントハンドラーになりますが、実際に発火するのはそのイベントが at に指定された DOM、すなわち `.ok-btn` からバブリングしてきたときだけです。すなわちこのメソッドは実質的に .ok-btn のイベントハンドラーのように働きます。

Capsid.js はこのように、JavaScript の class 構文に decorator を記述することで、ローカルな作用を意識したイベント駆動プログラミングを自然に行う事が出来るようにするためのツールです。

## Capsid.js は軽量

Capsid.js は非常に軽量です。サイズはわずか 1.7KB しかありません。Capsid.js は実質的に上のような記述を可能にするための糖衣構文のようなものです。複雑な runtime 実装などは一切持っていません。
