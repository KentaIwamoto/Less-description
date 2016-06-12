# Less.js 入門

友達の書いた記事が素晴らしいので共有します。

## プログラミングにおいて大事なこと
- KISS (Keep It Simple, Stupid)
- DRY (Don't Repeat Yourself)

## CSSの現状
- `a:hover` カーソルが上に来た時にだけ作用する
- `a:before`,`a:after` 要素前後に擬似的に要素を作り出しCSSを適用できる
- `calc` 単位の異なる数値を評価段階で計算してくれる機能
- `@media` 表示される端末によってCSSの表示を動的に変更する機能

プログラマブルな機能が追加されていく一方で要素を指定する方法は未だ原始的なものだけである。

- `.class`,`#id`,`tag` 通常の指定方法
- `tag.class` tagが示す要素の中でclassをclass属性として持つ要素のみに適用する
- `tag1,tag2`,`.class1,.class2` tag1とtag2両方に適用する。
- `tag tag1`,`tag .class`,`.class1 .class2` 並べることで範囲を左から順に絞って適用する
- `tag > tag`,`tag > .class`,`.class1 > .class2` 直下の要素のみに適用する
- `tag + tag`,`tag + .class`,`.class1 + .class2` 同じ階層の要素のみに適用する
- `tag[attributte="value"]` tagの中でattirbute属性の値がvalueとなるもののみに適用する

この原始的な記述方法のままだと以下のような問題が発生する。
1. ネストが深いドキュメントに対して局所的にCSSを適用させる場合、冗長な指定子が必要になる
(`#main div[role="main"] div.middle>span.dummy+em`)
2. １のようなものが大量にスタイルシートに発生して地獄絵図と化して、収集がつかなくなる。
3. 地獄絵図の中から的確にThemeを探すのは困難(もしくは、表にまとめて規約にする)
4. Themeが統一できていないので微妙に違う配色、ボーダーのデザインがなんか違う。


## 解決方法の提案
LessやSassと言ったCSSにコンパイルする処理系の登場
1. 変数の導入
2. 関数の導入
3. ネストしたCSSの記述

1,2によってDRYが、3によってKISSを実現することができる

### 1. 変数の導入

`@`で始めると変数として認識される。定義の仕方はCSSと同じである。また、デフォルトで四則演算が演算がサポートされている。もちろんカラーコード同士の足し算も可能である。

使用例
```
@color: #4D926F;

#header {
  color: @color;
}
h2 {
  /*少しだけ色を明るくする。*/
  color: @color + #111111;
}
```

テーマカラーの設定例(あとで `@import`して使う)
```
@pt-blue-1: #006699;
@pt-blue-2: #005588;

@pt-green-1: #339933;

@pt-white-1: #FAFAFA;
@pt-white-2: #EEEEEE;
@pt-white-3: #CCCCCC;

@pt-black-1: #AAAAAA;
@pt-black-2: #555555;

@pt-invisible: rgba(0,0,0,0);
```

### 2. 関数の導入

Mixinという。`.`と`(変数名)`で挟むことで関数を定義できる。また、デフォルト値も指定できる。

使用例
```
.rounded-corners (@radius: 5px) {
  -webkit-border-radius: @radius;
  -moz-border-radius: @radius;
  -ms-border-radius: @radius;
  -o-border-radius: @radius;
  border-radius: @radius;
}

#header {
  .rounded-corners;
}
#footer {
  .rounded-corners(10px);
}
```

こうすることで、今後ベンダープレフィックスを使用せずに`.rounded-corners`と書くだけで良くなる。
また、複数の設定をまとめることができるので、borderの管理などで活用できる。

### 3. ネストしたCSSの記述

`{}`の中にネストさせて書くことで要素以下の設定として認識される。

使用例
```
#header {
  h1 {
    font-size: 26px;
    font-weight: bold;
  }
  p { font-size: 12px;
    a { text-decoration: none;
      &:hover { border-width: 1px }
    }
  }
}
```

`&:`は親要素を指してる。ここでは`a`を表す。

### その他の便利な機能
デフォルトでいろいろなMixinが定義されている。
- `lighten`,`darken` 明暗を調節できる。
(`lighten(@color, 10%)`)
- `ceil`,`floor`　有名な天井関数と床関数
(`ceil(2.4) //=> 3`)

JavaScriptを内部展開して実行できる。
バッククオートで囲むことで展開される。

```
@var: `"hello".toUpperCase() + '!'`;
```
展開結果
```
@var: "HELLO!"
```

## Less.jsをおすすめする理由
CSSへのコンパイルを事前にしなくても、jsライブラリを導入することで、描画時にコンパイルしてくれる。（Sassは非サポート、しかし、その分、文字列同士の演算が可能だったりインデントベースの記述が可能になるなど、機能が豊富)

```
<script src="less.js" type="text/javascript"></script>
```

## 実例
以下は、あるウェブアプリのヘッダーとフッターを定義した.lessファイルです。

```
@pt-blue-1: #006699;
@pt-blue-2: darken(@pt-blue-1,10%);

@pt-green-1: #339933;

@pt-white-1: #FAFAFA;
@pt-white-2: darken(@pt-white-1,10%);
@pt-white-3: darken(@pt-white-1,20%);

@pt-black-1: #AAAAAA;
@pt-black-2: darken(@pt-black-1,60%);

@pt-invisible: rgba(0,0,0,0);

.pt-border-solid (@color: @pt-black-1) {
  border: solid 1px @color;
}

body{
  background-color:@pt-white-2;
  header,footer{
    nav{
      box-shadow:1px 0px 2px;
    }
    .navbar-default, .navbar-inner{
      background-color: @pt-blue-1;
      .pt-border-solid(@pt-blue-2);
      .navbar-brand, .nav > li > a{
         color: @pt-white-2;
         text-shadow: 0 -1px 0 rgba(0, 0, 0, 0.25);
         &:hover, &:focus{
            color: @pt-white-3;
         }
      }
    }
  }
  header{
    .breadcrumb{
      background-color:@pt-white-1;
      .pt-border-solid;
    }
  }
}
```

## 参考
- [Less.js(日本語)](http://less-ja.studiomohawk.com/#about)
- [WebWordスタイルシート入門](http://www.webword.jp/cssguide/basic/)
- [とほほのLess入門](http://www.tohoho-web.com/ex/less.html)

## 著作権表示
Copyrights (C) 2016 TANIGUCHI Masaya.

このドキュメントはCC0のもとでライセンスされています。


[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png "CC0")](http://creativecommons.org/publicdomain/zero/1.0/deed.ja)
