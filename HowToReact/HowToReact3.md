# state について

先ほどは props について説明しました。props は親コンポーネントから受け渡されるデータのことを指します。

これに対して state は各コンポーネントが持っている状態を表します。

例えば

```jsx
<button onClick={~~this.state.openの真偽値を反転する関数~~}>
    {this.props.label}
</button>
<Dialog open={this.state.open} />

```

というコンポーネントがあったとします。このコンポーネントは内部的な振る舞いとして、「ボタンを押下されると、ダイアログを開く」I/O を持ちます。  
対して、外部的な振る舞いとして「親から渡される props に従って、ボタンのラベルを変更する」I/O を持ちます。  
このコンポーネントを使うとき、親から見て、「ボタンを押下されると、ダイアログを開く」というロジックはコンポーネント内に収められており、変更出来るのはボタンのラベルだけです。  
このように、state を用いて内部的に状態を制御することによって、振る舞いをコンポーネントの中だけで完結させることができます。

# 成果物

<img src='./img/HowToReact/reactTour3.gif' />

## 初期値の設定

props の初期値の設定 (defaultProps) は必須ではありませんでしたが、state を扱う場合は初期値の設定が必須です。state の初期化には 2 通りの方法があります。

### constructor の中で初期化するパターン

```js
class TestComponent extends Component {
    constructor(props) {
        super(props);
        this.state = { ...略 };
    }
}
```

### class の中で初期化するパターン

```js
class TestComponent extends Component {
    state = { ...略 };
    // ...略
}
```

## state の実装例

ここで HRC コンポーネントを編集し、state を実装します。

```jsx
// HRC.js
import React, { Component } from 'react';

class HRC extends Component {
    constructor(props) {
        super(props);
        this.state = {
            text: 'Hello',
            bool: false,
        };
    }

    render() {
        const { text, bool } = this.state;
        const subText = bool ? 'on' : 'off';
        return (
            <div>
                <div>{text}</div>
                <div>{subText}</div>
            </div>
        );
    }
}

export default HRC;
```

これで state が実装されました。

```js
this.state = {
    text: 'Hello',
    bool: false,
};
```

上記の部分で、state に text, bool を設定しています。

state へのアクセスは props と同様であるため、

```
const { text, bool } = this.state;
```

にて取り出すことができます。

## state の変更

state の変更は React において重要なイベントです。state の変更は代入文でなく、setState メソッドを介して行う必要があります。  
※ 代入文で行うこともできますが React が state を管理できなくなるため、setState で行う必要があります。

それでは、ボタン押下で state が変更されるように HRC.js を書き換えます。

```jsx
// HRC.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class HRC extends Component {
    constructor(props) {
        super(props);
        this.state = {
            text: 'Hello',
            bool: false,
        };
        this._handleClick = this._handleClick.bind(this);
    }

    _handleClick() {
        this.setState({ bool: !this.state.bool });
    }

    render() {
        const { text, bool } = this.state;
        const subText = bool ? 'on' : 'off';
        return (
            <div>
                <button onClick={this._handleClick}>{text}</button>
                <div>{subText}</div>
            </div>
        );
    }
}

export default HRC;
```

以上の変更で、ボタン押下によって、on, off のテキスト変更がされるようになります。
また、state が変更されると再び、render メソッドが実行されます。このような React 独自のライフサイクルメソッドなどについては<a href="./HowToReact4">次ページ</a>にて解説いたします。

### 構文説明

#### setState の使い方

state の変更は setState メソッドによって実行することができます。

```
this.setState( Object );
```

という構文です。ただし、this は現在のコンポーネントを指し示す必要があります。

#### setState の挙動

上記サンプルでは、

```
this.state = {
    text: 'Hello',
    bool: false,
};
```

このように state が設定されています。これを変更したい場合には、

```
this.setState({
    text: 'Evening',
    bool: true,
});
```

とします。すると下記のように state が変更されます。

```
        変更前                         変更後
    this.state.text = "Hello"   this.state.text = "Evening"
    this.state.bool = false     this.state.bool = true
```

単一のプロパティのみを変更したい場合は、

```
this.setState({ bool: true });
```

とします。text は setState メソッドに含めていないため、text は現在の値のまま、bool にのみ変更がかかります。

初期状態から、this.setState({ bool: true });した場合、state は下記のようになります。

```
        変更前                         変更後
    this.state.text = "Hello"   this.state.text = "Hello"
    this.state.bool = false     this.state.bool = true
```

#### this.func = this.func.bind(this)

コンストラクター内で下記の記述があります。

```jsx
constructor(props){
    ...略
    this._handleClick = this._handleClick.bind(this);
}
```

以下の条件に全て当てはまる場合は、bind(this)する必要があります。

-   React のライフサイクルメソッド（render など）ではない
-   メソッド内でこのコンポーネントを参照している（this Word など）。
-   onClick などのイベントハンドラでメソッドが呼ばれる (HRC.js の \_handleClick など）

これを使うことによって、この関数（上記の例では\_handleClick メソッド）内の this コンテキストがこのコンポーネントに固定されます。

#### onClick

イベントに関数を割りあてるためには onClick などの属性を DOM にあてます。
上記サンプルでは、

```
<button onClick={this._handleClick}>{text}</button>
```

となっており、button のクリックイベントに \_handleClick メソッドがバインドされます。

## props を利用して子コンポーネントから親コンポーネントの state を変更させる

オブジェクトは参照渡しである性質を利用して、親コンポーネントの state を子コンポーネントから変更させることができます。

親コンポーネントである App.js のコードを下記のように変更します。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM 'react-dom';
import HRC from './HRC';

class App extends Component {

    constructor(props) {
        super(props);
        this.state = {
            text: 'off',
        };
        this._handleChange = this._handleChange.bind(this);
    }

    _handleChange(value) {
        this.setState({ text: value });
    }

    render() {
        return (
            <div>
                <HRC handleChange={this._handleChange}>
                    this is this.props.children
                </HRC>
                <div>App/{this.state.text}</div>
            </div>
        );
    }
}

ReactDOM.render(
    <App />,
    document.getElementById('content'),
);
```

次に HRC.js のコードを下記のように変更します。

```jsx
// HRC.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class HRC extends Component {
    constructor(props) {
        super(props);
        this.state = {
            text: 'Hello',
            bool: false,
        };
        this._handleClick = this._handleClick.bind(this);
    }

    _handleClick() {
        const { bool } = this.state;
        this.setState({ bool: !bool });
        this.props.handleChange(!bool ? 'on' : 'off');
    }

    render() {
        const { text, bool } = this.state;
        const subText = bool ? 'on' : 'off';
        return (
            <div>
                <button onClick={this._handleClick}>{text}</button>
                <div>{subText}</div>
            </div>
        );
    }
}
HRC.defaultProps = {
    handleChange() {},
};
HRC.propTypes = {
    handleChange: PropTypes.func,
};

export default HRC;
```

以上で、子コンポーネント（HRC) の持つボタンを押下することで親コンポーネントの state が変更（「App/off」の off が on に切り替わる）されるようになります。

### 構文解説

親コンポーネント（App)は、子コンポーネント（HRC)に handleChange という属性で、this.\_hanldleChange メソッドを渡しています。
この時、this.\_handleChange 内で利用される setState の this コンテキストを App にしたいため、

```
constructor(props) {
    ︙
    this._handleChange = this._handleChange.bind(this);
}
```

が必要になります。

これを受けた子コンポーネント（HRC)は

```
_handleClick() {
    ︙
    this.props.handleChange(!bool ? 'on' : 'off');
}
```

と受け取った handleChange（実体は親コンポーネントのもつ handleChange メソッド）という props を実行することで、親コンポーネントの\_handleChange を実行し、親コンポーネントの state を変更することができます。
