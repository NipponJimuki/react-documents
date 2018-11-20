# 入力フォーム管理について

React における入力フォームの管理方法は 2 種類あります。

-   state として管理する（推奨）
-   ref から直接取得する

どちらの方法でも値を取得することは可能なのですが、React の概念では入力状態も state で管理する指針のため、ref から取得する形式はある意味アンチパターンと取れます。そのため state として管理する方法を推奨します。  
<br>
本項では、state としての管理方法と、ref による管理方法の両方を説明します。

# 成果物

<img src='../img/HowToReact/reactTour6.gif' />

# state による入力フォーム管理

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            text: '',
            result: '',
        };
        this._handleChange = this._handleChange.bind(this);
        this._onSend = this._onSend.bind(this);
    }
    _handleChange(e) {
        this.setState({ text: e.target.value });
    }
    _onSend() {
        this.setState({ result: this.state.text });
    }
    render() {
        return (
            <div>
                <input type="text" onChange={this._handleChange} />
                <button onClick={this._onSend}>送信</button>
                <div>
                    <div>送信値</div>
                    <div>{this.state.result}</div>
                </div>
            </div>
        );
    }
}
ReactDOM.render(<App />, document.getElementById('content'));
```

こちらのコードではまず、state に

```jsx
this.state = {
    text: '', //フォーム入力内容を管理する
    result: '', //送信値に紐付いた値
};
```

を設定します。  
次に input に onChange イベントとして

```jsx
_handleChange(e) {
    this.setState({ text: e.target.value });
}
```

をセットします。  
引数の e は React でラップされたクロスブラウザ対応している
<a href="https://facebook.github.io/react/docs/events.html" target='_blank'>SyntheticEvent</a> です。通常の EventObject と同じように扱うことができます。  
そのため、入力された値は e.target.value にて取得することができます。
onChange に bind しているため、入力がある都度 state を更新するメソッドとなります。

### イベントの捕捉： bind

以下の条件に全て当てはまる場合は、bind(this)する必要があります。

-   React のライフサイクルメソッド（render など）ではない
-   メソッド内でこのコンポーネントを参照している（this Word など）。
-   onClick などのイベントハンドラでメソッドが呼ばれる

ということを<a href="./#/HowToReact3">HowToReact3</a>では紹介しました。そのため、上記コードでは onChange, onClick に指定されているメソッドを、bind します。

```js
constructor(){
    ・・・・
    this._handleChange = this._handleChange.bind(this);
    this._onSend = this._onSend.bind(this);
}
```

# ref で値を取得する

ref を使うことで値を取得することもできます。ref は html における id に近い機能です。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            result: '',
        };

        this.textInput = React.createRef();
        this._onSend = this._onSend.bind(this);
    }

    _onSend() {
        const result = this.textInput.current.value;
        this.setState({ result });
    }

    render() {
        return (
            <div>
                <input type="text" ref={this.textInput} />
                <button onClick={this._onSend}>送信</button>
                <div>
                    <div>送信値</div>
                    <div>{this.state.result}</div>
                </div>
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

先ほどとは異なり、こちらの方法は state での管理を行う必要がないため onChange メソッドはありません。代わりに ref を設定しています。  
ref は

```
_onSend() {
    const result = this.textInput.current.value;
    this.setState({ result });
}
```

のように

```
    this.textInput.current
```

によって ref が指定されているノードへアクセスすることができます。  
例えば、input 要素の value を取得したい場合、

```
    this.textInput.current.value
```

にて値を取得することができます。

## Stateless Functional Component について

React には、Stateless Functional Component（以下 SFC）というコンポーネントがあります。名前の通り、状態を持たない（stateless）関数型のコンポーネントになります。今までの React Component では、render メソッドの中で描画したい DOM を返していましたが、SFC では、return 文の中に直接書くことができます。  
メリット

-   与えられた props を元にレンダリングするだけなので、可読性が高い
-   ライフサイクルメソッドを持たないので速い

デメリット

-   componentDidMount 等のライフサイクルメソッドが定義できない
-   state や外部ライブラリのインスタンス管理ができない
-   ライフサイクルメソッドが必要になったらクラスベースでの書き直しが必要になる

メンテナンス性の向上にも繋がるので、state 管理の不要なものについて、SFC で記述することを推奨します。  
試しに、App.js の

```
<div>
    <div>送信値</div>
    <div>{this.state.result}</div>
</div>
```

を SFC として切り出してみましょう。

HRC.js を下記のように変更します。

```jsx
// HRC.js
import React from 'react';

const Result = props => (
    <div>
        <div>送信値</div>
        <div>{props.result}</div>
    </div>
);

Result.defaultProps = {
    result: '',
};

export default Result;
```

HRC.js を呼び出す App.js にも変更を加えます。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import Result from './HRC.js';

class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            result: '',
        };

        this.textInput = React.createRef();
        this._onSend = this._onSend.bind(this);
    }

    _onSend() {
        const result = this.textInput.current.value;
        this.setState({ result });
    }

    render() {
        return (
            <div>
                <input type="text" ref={this.textInput} />
                <button onClick={this._onSend}>送信</button>
                <Result result={this.state.result} />
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

## 構文解説

```js
const Result = (props) => {
```

今まで使ってきた React Component は class であったため、this.props の形で props にアクセスしていました。しかし、SFC は関数であるため props を直接、引数として扱えます。  
また、**(props)** の部分を **({ result })** と記述することもできます。引数として受け取ると同時に、props オブジェクトから result というプロパティを取り出しています。分かりやすく書くと、

```js
const props = {
    result: '入力値',
};
const { result } = props;
```

になります。

先に props を変数に展開するメリットとしては、

-   関数内で変数の宣言が不要になること
-   return 文の省略が可能になること

があげられます。

### 補足

React チームは以前から、クラスという概念がコードの見通しを悪くしており、なるべく SFC でコンポーネントを書くようにと推奨していました。  
React v16.7.0-alpha.0 から React Hooks という新しい概念が追加されました。  
これを使うことで、全てのコンポーネントを関数として記述でき、クラスを用いずともライフサイクルメソッド（一部）や state の利用ができます。
React Hooks の登場によって、Stateless Functional Component ではなく **Function Components** という呼び方に今後変わっていきますので注意してください。
