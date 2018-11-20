# スタイリングについて

React では従来のように CSS を使ったスタイリングの他、

-   CSS in JS (CSS も JavaScript で書く方法）
-   CSS Module (グローバルな CSS の適用範囲を制限する）

でスタイリングを行っております。本項では、通常の CSS でスタイリングする方法と CSS in JS でスタイリングする方法を紹介します。  
Web 開発基盤では CSS in JS を推奨しています。

# 成果物

<img height= 150 width= 600 src='../img/HowToReact/reactTour5.gif' />

## 通常の CSS で書くメリット・デメリット

-   すでに CSS を書くことができるのであれば学習コストが要らない
-   グローバルに展開されるため、命名規則・ルールなどを競合が起こらないような設計にしなければならない

## CSS in JS で書くメリット・デメリット

-   ほんの少しの学習が必要
-   mediaQuery が使えず、疑似要素も一部しかサポートされていない
-   影響をあたえる範囲が極めて局所的なため、命名規則・ルールなどで競合が起こりにくい
-   コーディングの文字数は増加するが、直感的に使用できるため保守面では非常に強い

## CSS Module で書くメリット・デメリット

-   CSS in JS では扱えないセレクタを扱えるようになる
-   ピュアな CSS そのものを使うことができる
-   カスケーディングの構造を正す必要がある場面もある

## CSS in JS を使用したスタイリング

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

const styles = {
    helloDiv: {
        backgroundColor: '#008080',
        color: 'white',
        fontFamily: 'Roboto',
        height: '50px',
        margin: '5rem',
        lineHeight: '50px',
        textAlign: 'center',
    },
    helloSpan: {
        fontSize: '30px',
        fontStyle: 'italic',
        margin: '1rem',
    },
};

class App extends Component {
    render() {
        return (
            <div style={styles.helloDiv}>
                Hello
                <span style={styles.helloSpan}>React</span>
                World!
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

以上が CSS in JS でのスタイリング方法になります。styles は App.js の中に定義されていますが、styles.js というファイルを作ってそれを読み込むようにすることもできます（こちらの方法については LibraryWeb アプリツアーの <a href="./#/styling">DevTourStyling</a> にて解説します）。

スタイリングは CSS ではなく、JavaScript オブジェクトとして行います。

```js
const styles = {
    helloDiv: {
        backgroundColor: '#008080',
        color: 'white',
        fontFamily: 'Roboto',
        height: '50px',
        margin: '5rem',
        lineHeight: '50px',
        textAlign: 'center',
    },
    helloSpan: {
        fontSize: '30px',
        fontStyle: 'italic',
        margin: '1rem',
    },
};
```

この時の注意点としては、

-   オブジェクトとして作成すること。
-   ハイフン（-)を持つプロパティ名はキャメルケースに書き換えること
-   プロパティ値は文字列で書くこと

です。

### CSS in JS に styled-components を使う

次は styled-components を利用した CSS in JS を紹介します。
上で紹介した、コンポーネントに対して style props を渡す方法では出来なかった、疑似要素の使用が可能になります。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import styled from 'styled-components';

class App extends Component {
    render() {
        return (
            <HelloDiv>
                Hello
                <HelloSpan>React</HelloSpan>
                World!
            </HelloDiv>
        );
    }
}

const HelloDiv = styled.div`
    background-color: #008080;
    color: white;
    font-family: Roboto;
    height: 50px;
    margin: 5rem;
    line-height: 50px;
    text-align: center;
    transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1, 100ms);
    :hover {
        transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 100ms;
        background-color: #00a0a0;
    }
`;

const HelloSpan = styled.span`
    font-size: 30px;
    font-style: italic;
    margin: 1rem;
`;

ReactDOM.render(<App />, document.getElementById('content'));
```

以上で styled-components の導入は完了です。これによって hover を追加することができました。

```js
import styled from 'styled-components';
```

と記述することで、styled-components をインポートします。

先ほどはコンポーネントに対して、style という props を渡して、スタイルの適用を行なっていました。それに対して、styled-components ではコンポーネント自体をラッピングすることでスタイルを適用します。

```js
const HelloDiv = styled.div`
    background-color: #008080;
    color: white;
    ...略;
`;
```

先ほどはコンポーネントに対して、style という props を渡して、スタイルの適用を行なっていました。それに対して、styled-components ではコンポーネント自体をラッピングすることでスタイルを適用します。

```js
const HelloDiv = styled.div`
    background-color: #008080;
    color: white;
    ...略;
`;
```

ここで、\`[文字列]\`という見慣れない記法が出てきました。これはテンプレートリテラルという JavaScript の新しい機能の 1 つです。\`\`で囲まれた範囲を文字列として扱います。

```js
const num = 11;
const text = `num is ${num} ! 改行も
できるよ！`;
console.log(text);
// num is 11 ! 改行も
// できるよ!
```

テンプレートリテラルでは、上記のように文字列に変数の値を含めたり、改行コードを含めたりすることができます。

また、コード中の styled.div\`[文字列]\` のようにテンプレートリテラルは関数の引数として使うことができます。
この記法をタグ付きテンプレートリテラルといいます。  
簡潔に述べると、関数の新しい書き方の 1 つで、\`\` の中の文字列を引数として styled.div 関数を実行しています。今まで出てきた、func() との違いは実行時の引数の扱い方が異なる点です。(気になった方は、タグ付きテンプレートリテラルについて調べてみてください）

ここで定義された HelloDiv は変更前の \<div\> タグと同じように、\<HelloDiv\> の形で利用できます。また、style オブジェクトを作成した時と違い、記法がより CSS 本来のものに近くなっていることに注意してください。

```js
:hover {
    transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 100ms;
    background-color: #00a0a0;
}
```

このように、通常の CSS と同じように擬似要素を使うことができます。

hover の他にも

-   active
-   focus
-   @media

などを使用することができます。

## styled-components に props を適用する

styled-components で定義されたコンポーネントでは、props を扱うこともできます。

```jsx
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import styled from 'styled-components';

class App extends Component {
    render() {
        return (
            <HelloDiv color="blue" height={90}>
                Hello
                <HelloSpan>React</HelloSpan>
                World!
            </HelloDiv>
        );
    }
}

const HelloDiv = styled.div`
    background-color: #008080;
    color: ${props => props.color};
    font-family: Roboto;
    height: ${props => `${props.height}px`;
    margin: 5rem;
    line-height: 50px;
    text-align: center;
`;

const HelloSpan = styled.span`
    font-size: 30px;
    font-style: italic;
    margin: 1rem;
`;

ReactDOM.render(<App />, document.getElementById('content'));
```

上記のように、styled-components でラップされたコンポーネントに対して与えられた props を \${(props) => プロパティの値} という関数形式で利用できます。

## CSS Module を使用したスタイリング

CSS Module を使用するとほぼほぼ従来と同じようなスタイリング方法を行うことができます。ただし、従来と異なる点としては、クラス名はグローバルに影響を及ぼさず、ローカルに影響を及ぼす。  
また、要素から始めるセレクタを使う場合には効力を発揮しないため注意してください。

-   ローカルのみに影響を及ぼす

```css
  .valid div {・・・・}
```

-   グローバルに影響を及ぼす

```css
div {・・・}
div .valid {・・・}
```

src フォルダ内に style.css を作成します。
そして、下記コードを記述します。

```css
.helloDiv {
    background-color: #008080;
    color: white;
    font-family: Roboto;
    height: 50px;
    margin: 5rem;
    line-height: 50px;
    text-align: center;
    transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 200ms;
}
.helloDiv:hover {
    transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 100ms;
    background-color: #00a0a0;
}

.helloSpan {
    font-size: 30px;
    font-style: italic;
    margin: 1rem;
}
```

次に App.js を編集します。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import styles from './style.css';

class App extends Component {
    render() {
        return (
            <div className={styles.helloDiv}>
                Hello
                <span className={styles.helloSpan}>React</span>
                World!
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

CSS モジュールを使うためには CSS を読み込む必要があります。

```js
import styles from './style.css';
```

読み込む際には、拡張子（.css)を忘れずにつけてください。
cssModule はクラス名を一意のものに書き換える仕組みです。そのため、CSS Module を使う場合は、

```jsx
<div className={styles.helloDiv}>
    Hello
    <span className={styles.helloSpan}>React</span>
    World!
</div>
```

このように、className={styles.helloDiv}と、読み込んだプロパティを使用してクラスを付与します。
また、この時でてきた className という属性は、JSX におけるクラス名を指定するための属性で CSS Module の仕様とは関係ありません。

言い換えると、class 属性は React では className という名称で指定するということになります。
