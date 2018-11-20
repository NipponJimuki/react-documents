# 入力フォームを扱う

前のページでは簡単なアプリケーションを作成しました。  
今回は入力値を保存するアプリケーション（Todo アプリもどき）を作成します。  
フォームに入力した値を Redux で管理し、追加ボタン押下で登録という機能を実装します。

# 成果物

<img src='../img/HowToRedux/reduxTour3.gif' />

## 入力フォームを含むコンポーネントの作成をする

Web 開発基盤では formik の利用を推奨していますが、ここでは、formik を使用しない方法で、新たな SmartComponent を作成します。
components 配下に AddItem.js を作成し、以下のコードを作成します。

```jsx
// components/AddItem.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class AddItem extends Component {
    static defaultProps = {
        textChangeAction() {},
        addItemAction() {},
        textValue: '',
        items: [],
    };

    _onChange = e => this.props.textChangeAction(e.target.value);
    _onClick = () => this.props.addItemAction(this.props.textValue);

    render() {
        const { textValue, items } = this.props;
        return (
            <>
                <input type="text" onChange={this._onChange} value={textValue} />
                <button onClick={this._onClick}>追加</button>
                {items.join('/')}
            </>
        );
    }
}
AddItem.propTypes = {
    textChangeAction: PropTypes.func,
    addItemAction: PropTypes.func,
    textValue: PropTypes.string,
    items: PropTypes.array,
};
export default AddItem;
```

このコンポーネントが受け取る props は

-   textChangeAction: フォームに入力された値を管理 textValue に流し込む Action
-   addItemAction: items に値を追加する Action
-   textValue: フォームに入力されている値
-   items: 追加ボタンを入力した時の状態を保存している値（Todo アプリでいう TodoList に相当）

となっています。  
次に今までと同様に index.js を編集します。

```jsx
// components/index.js
export { default as Main } from './Main';
export { default as AddItem } from './AddItem';
```

### コードの説明

先ほど追加したソースコードの解説を行います。
action や reducer はまだ実装していませんが、これらは後述します。

#### input

```jsx
_onChange = e => this.props.textChangeAction(e.target.value);

<input type="text" onChange={this._onChange} value={textValue} />;
```

input エレメントでは、ユーザがフォーム（input)に入力をする度に \_onChange メソッドが実行されます。

onChange では、this.props.textChangeAction という ActionCreator が実行され、引数（e.target.value）には現在の入力値が含まれています。  
textChangeAction に e.target.value が常に渡されることで、textValues は常に現在の入力値を保持することができています。

#### button

```jsx
_onClick = () => this.props.addItemAction(this.props.textValue);

<button onClick={this._onClick}>追加</button>;
```

button エレメントでは、ボタンを押下することで \_onClick メソッドが実行されます。

\_onClick では、this.props.addItemAction という ActionCreator が実行されます。  
この時の引数（this.props.textValue)には現在の入力値が含まれています。
追加ボタンを押下することで、押した時点でのフォームの入力値が this.props.items に追加されます。

## Container の作成

次は AddItem のコンテナーを作成します。
containers 配下に AddItem.js を作成し、以下のコードを作成します。

```js
// containers/AddItem.js
import { connect } from 'react-redux';
import { AddItem } from '../components';
import { textChange, addItem } from '../actions';

const mapStateToProps = ({ addItemState }) => ({
    ...addItemState,
});

const mapDispatchToProps = dispatch => ({
    textChangeAction(value) {
        dispatch(textChange(value));
    },
    addItemAction(value) {
        dispatch(addItem(value));
    },
});

export default connect(
    mapStateToProps,
    mapDispatchToProps,
)(AddItem);
```

先ほどのページでは reducer が 1 つだったので、state をそのまま、mapStateToProps で使っていました。  
まだ reducer は作成していませんが、今回から複数の reducer が存在します。その場合、

```js
const mapStateToProps = ({ addItemState }) => ({
    ...addItemState,
});
```

このように store から 特定の state のみを取得して流し込むということをします。  
store で管理する状態を Container 単位で切り分けておくと管理しやすくなります。

addItemState には、AddItem Component で使う textValue と items という値がオブジェクトのキーバリューの形で保存されています。  
それをスプレッドオペレーターでそのまま AddItem Component に渡しています。

同様に index.js も修正します。

```js
// containers/index.js
export { default as Main } from './Main';
export { default as AddItem } from './AddItem';
```

### mapStateTopProps の引数について

引数の中に突然でてくる中カッコ（オブジェクトリテラル）ですが、これは分割代入（Destructuring assignment）という構文になります。  
厳密には、引数内で分割代入とオブジェクトのプロパティ省略記法という 2 つの糖衣構文を使っています（関数内でのオブジェクト取得を分割代入と呼ぶわけではありません）。  
分割代入は、配列とオブジェクトに対して使える構文で、値あるいはプロパティを取り出して別個の変数として定義します。

```js
// 例
const obj = { id: 1, name: 'hoge', age: 22 };
const str = '2018/10/1';

const {
    id,
    age,
    name: userName, // name を 取り出して userName という変数名で定義している
} = obj;
const newObj = { id, age, userName }; // プロパティ省略記法
const [year, month, day] = str.split('/'); // splitは配列を返すので、返却値に対して分割代入を行なっている

console.log(id, userName, age); // 1, 'hohe', 22
console.log(newObj); // { id: 1, age: 22, userName: 'hoge' }
console.log(year, month, day); // 2018, 10, 1
```

分割代入の特徴は、関数実行後の返却値や関数の引数内でも利用できるところにあります。

```js
const [year, month, day] = str.split('/'); // 関数の返却値で分割代入・・・①
const mapStateToProps = ({ addItemState }) => ({
    addItemState,
}); // 関数の引数内で分割代入・・・②

// ①
const str = '2018/10/1';
const array = str.split('/');
const [year, month, day] = array;
console.log(year, month, day); // 2018, 10, 1

// ②
const store = {
    changePowerState: { power: false },
    addItemState: { textValue: '', items: [] },
};
const func = store => {
    const { changePowerState } = store;
    return { changePowerState };
};
console.log(func(store)); // { power: false }
```

① と ② のコードは同等のことをしており、定義した変数には同じ値が入っています。  
このように、ESNext を使いこなすことで、従来よりも少ない記述量でコードを書くことができます。

## ActionCreator の作成

### actionTypes の追加

新しく ActionCreator を作成するためにアクション定数を追加します。

```js
// actions/actionTypes.js
export const CHANGE_POWER_STATE = 'CHANGE_POWER_STATE';
export const TEXT_CHANGE = 'TEXT_CHANGE';
export const ADD_ITEM = 'ADD_ITEM';
```

続いて、Action を作成します。  
actions 配下に新しく addItem.js を作成し、下記のコードを作成します。

```js
// actions/addItem.js
import { TEXT_CHANGE, ADD_ITEM } from './actionTypes';

export const textChange = payload => ({
    type: TEXT_CHANGE,
    payload,
});

export const addItem = payload => ({
    type: ADD_ITEM,
    payload,
});
```

payload は Action の引数で、textChange には textValue に渡される文字列が入っており、
addItem には items に追加される textValue の文字列が入ってきます。

新しくファイルを作成したので、index.js を編集します。

```js
// actions/index.js
export * from './main';
export * from './addItem';
```

#### ActionCreator の引数

payload は、Action の引数となり、reducer あるいは middleware での処理に使用します。  
Redux において、ActionCreator は type プロパティを必ず持ち、識別子などを使用したい場合は payload というプロパティに値をいれます。  
その他の値は meta というプロパティに格納することが望ましいです。  
ルールが定められているというわけではありませんが、どちらかと言えばお作法に近いです（役割が明確になるという点ではメリットです）。

| プロパティ名 | 必須 |                  説明                   |
| :----------: | :--: | :-------------------------------------: |
|     type     |  ○   | Action を識別するためのユニークな文字列 |
|   payload    |      |    state を更新するために必要な引数     |
|     meta     |      |              その他の引数               |

## reducer の定義

AddItem コンポーネントで使用する textValue と items を更新するための reducer を定義します。
reducer に新しく addItem.js というファイルを作成し、下記コードを作成します。

```js
// reducers/addItem.js
import { TEXT_CHANGE, ADD_ITEM } from '../actions/actionTypes';

const initialState = {
    textValue: '',
    items: [],
};

export const addItemState = (state = initialState, action) => {
    const { type, payload } = action;

    switch (type) {
        case TEXT_CHANGE: {
            return {
                ...state,
                textValue: payload,
            };
        }
        case ADD_ITEM: {
            return {
                ...state,
                items: [...state.items, payload],
            };
        }
        default: {
            return state;
        }
    }
};
```

state の更新には、action の payload を利用します。  
これが action の payload を利用した state 更新の雛形となります。

### reducer の合成

reducers/index.js を修正し、main.js と addItem.js の reducer を合成して 1 つにまとめます。

```jsx
// reducers/index.js
import { changePowerState } from './main';
import { addItemState } from './addItem';
import { combineReducers } from 'redux';

const reducers = combineReducers({
    changePowerState,
    addItemState,
});

export default reducers;
```

reducer の合成には combineReducers を使用して、キーバリューで渡します。
これで作成される store の値は

```js
{
    changePowerState: { power: false },
    addItemState: { textValue: '', items: [] },
}
```

となります。  
糖衣構文を利用して reducer 名をそのまま state の名前にしていますが、
キー名を指定すれば、任意の state 名に変更することができます。

#### reducer の更新に関して

reducer の更新には

```js
return {
    ...state,
    textValue: payload,
};
```

このように、以前の state を必ず利用するようにします。
上記コードはスプレッドオペレーターと JavaScript のオブジェクトの特徴を活かした書き方になります。  
先ずはスプレッドオペレーターで以前の state を全て流し込んで state の復元をします。  
その上で、更新したい state だけを payload で上書きするということをしています。  
実際に実行されるコードでは以下のように値が返却されます。

```js
/**
 * store = { textValue: 'hog', items: ['foo', 'bar'] }
 * payload = 'hoge'
 */
return {
    textValue: 'hog',
    items: ['foo', 'bar'],
    textValue: 'hoge',
};

/**
 * 返却されるオブジェクトは
 * { textValue: 'hoge', items: ['foo', 'bar'] }
 */
```

前のツアーでスプレッドオペレーターはシャローコピーすることを説明しました。  
...state は以前の state の値が入っていますが、渡される値は新しく作成されたオブジェクトなので、参照渡しではなく値渡しになっています（オブジェクトは参照渡しです）。  
つまり、reducer を更新する際は、以前の state を破棄して新しい state を作成してコンポーネントに渡すということを
この return 文で行なっています。

<br />
いかにして値をイミュータブル（不変性）にするかが、フロントエンドアプリの設計において重要な鍵となります。

## App.js の変更

reducer を reducers/index.js で作成するようにしたので、store の作成をしている App.js を変更します。

```jsx
// App.js
import React from 'react';
import { render } from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import reducers from './reducers';
import { AddItem } from './containers';

const store = createStore(reducers);

// DOM出力
render(
    <Provider store={store}>
        <AddItem />
    </Provider>,
    document.getElementById('content'),
);
```

前のツアーでは Main コンポーネントを読み込んでレンダーしていましたが、今回は使わないので
替わりに AddItem コンポーネントをレンダーするようにします。

<br />
以上で作業は完了です。
