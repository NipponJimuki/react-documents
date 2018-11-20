# Middleware を扱う

今回の章では、HowToRedux2 と HowToRedux3 で作成したコンポーネントや Action を組み合わせて家電管理アプリケーションを作成します。  
具体的には、フォームにテキストを入力して追加ボタンを押すと、入力値が保存されるようにします（AddItem）。  
そして、保存された入力値にそれぞれ on / off という状態を持たせます（Main）。  
また、追加ボタンで家電を登録した際にフォームに入力されているテキストをリセットする機能を追加します。

# 成果物

<img width=700 src='../img/HowToRedux/reduxTour4.gif' />

# コンポーネントの呼称変更

先ほど便宜的に Main と指しましたが、登録された家電をそれぞれ表示するコンポーネントにこの名前は相応しくありません。  
リーダブルコードの観点においても、Main という名称はあまりよろしくないです。

<br />
そこで、MainではなくItemListという名称に改めることにします。

# Middleware の使用方法

Middleware では、複数のアクションの発行、他アクションへの伝播や情報の不可や改変などを行うことができます。  
今回は Middleware を AddItem と ItemList でそれぞれ使用します。
Middleware で実装するロジックは以下のとおりです。

-   AddItem
    -   新しい家電をセットする
    -   フォームの入力値をリセットする
    -   入力値がなければ処理を中断する
-   ItemList
    -   該当の家電のみ電源を変更する

## Middleware の役割

HowToRedux1 で Middleware は、ドメイン層であると紹介しました。  
Redux においてデータ（payload, meta)は Component を起点にして

```
Action -> Middleware -> Reducer
```

という順番にデータが流れていきます。この時、Middleware を経由しないパターンもあれば、Middleware でデータフローが止まってしまうパターンもあります。

主に、

-   データ整形
-   API 通信
-   非同期処理
-   他の Action を発行

という役割があります。Redux の性質上、ここに多くの処理を記述することになります。

# Middleware の作成

先にコードを作成し、コードの解説は後で行います。

## 家電を登録するロジックの作成

家電を登録する際に実行される addItem の Middleware を作ります。
middlewarers 配下に addItem.js を作成し、以下のコードを作成します。

```js
// middlewares/addItem.js
import { ADD_ITEM } from '../actions/actionTypes';
import { textChange as textChangeActionCreator } from '../actions';

const addItem = store => next => action => {
    switch (action.type) {
        case ADD_ITEM: {
            const { items, textValue } = store.getState().addItemState;
            if (!textValue) return false;
            const addedItems = [
                ...items,
                {
                    id: `item-index-of-${items.length}`,
                    name: textValue,
                    power: false,
                },
            ];
            action.payload = addedItems;

            store.dispatch(textChangeActionCreator(''));
            next(action);
            break;
        }

        default:
            next(action);
    }
};

export default addItem;
```

addItem の middleware では、まずフォームの入力値を if 文で判定しています。  
もし何も入力されていない空文字であれば、その時点で処理を中断して middleware の実行と reducer の更新を止めています。

<br />
入力値があれば、スプレッドオペレーターで現在の items の値を復元して、
id, name, power というキー値を持つオブジェクトを作り連想配列を作成しています。  
このキー値はそれぞれ

-   id : 家電のユニークな識別子
-   name : 家電名
-   power : 電源が入ってるかどうかの真偽値

を意味しています。  
新しく作成した items(addedItems)を action の payload に追加して next(action)で reducer を更新するということを行なっています。

## 家電の電源を変更するロジックの作成

続いて家電の電源を変更する itemList の Middleware を作ります。
同様に middlewares 配下に itemList.js を作成し、以下のコードを作成します。

```js
// middlewares/itemList.js
import { CHANGE_POWER_STATE } from '../actions/actionTypes';
import { addItem } from '../actions';

const changePowerState = store => next => action => {
    const { type, payload } = action;

    switch (type) {
        case CHANGE_POWER_STATE: {
            const { items } = store.getState().addItemState;
            const changedPowerItems = items.map((key, index) =>
                index === payload ? { ...key, power: !key.power } : key,
            );
            next(addItem(changedPowerItems));
            break;
        }
        default:
            next(action);
    }
};

export default changePowerState;
```

itemList の Middleware では、家電の電源の on /off を切り替えるロジックを

```js
const changedPowerItems = items.map((key, index) =>
    index === payload ? { ...key, power: !key.power } : key,
);
```

で実装しています。  
map 関数はプリミティブ型の配列についているメソッドになり、新しい配列を生成します。  
コールバックは配列の要素数だけ実行され、
action.payload（何番目の家電かを示す index）と items の index が一致したものがあれば、  
power の真偽値を反転させたオブジェクトを返却、それ以外は元のオブジェクトを返却
ということをしています。

#### map 関数について

JavaScript には配列を操作するための reduce や filter といった便利な関数がいくつかあります。  
map 関数もその中の 1 つで、第 1 引数にコールバックを受け取ることができ、コールバックは配列の要素数だけ実行されます。
下の例だと、コールバックは合計 5 回実行されます。

```js
// 例
const arr = [1, 2, 3, 4, 5];
const newArray = arr.map((key, index, array) => {
    return key * index
};

console.log(newArray); // [0, 2, 6, 12, 20]
```

コールバック内の引数の内訳は下記の表の通りです。

| 引数  |        説明        |
| :---: | :----------------: |
|  key  |  配列のカレント値  |
| index | 現在の配列の index |
| array |      元の配列      |

例えば、1 回目のコールバック時の値はそれぞれ

-   key : 1
-   index : 0
-   array : [0, 1, 2, 3, 4]

となっています。

## 構文解説

middleware を 2 つ作成しましたが、作成したコードにはどちらも

`const addItem = store => next => action => {・・・}`

という=>（アロー演算子）を複数使った見慣れない書式がでてきています。  
middleware の雛形となる書き方ですが、これは高階関数と呼ばれるもので、関数を返す関数です。  
今回の場合、middleware が実行される中で 3 回コールされています。

1 回目のコールで store が
2 回目のコールで next が
3 回目のコールで action が
それぞれ渡されており、3 回目のコール時に関数の中身が実行されます。

#### 高階関数について

高階関数は関数を返す関数と説明しましたが、これは実に多様な表現力を持っており、
様々な実装場面で応用することができます。

```js
// 例
const func = num1 => num2 => num1 + num2;

func(10)(100); // 110
```

とても簡単な例ですが、高階関数はこのように実行することができます。  
1 回目の実行（10)ではまだ値は返却されず、2 回目の実行（100)で演算が行われ値が返却されます。

<br />
C++や Haskell という言語にはカリー化という技術がありますが、JavaScript にも関数のカリー化というものがあり、
カリー化を実装する際にこの高階関数の考え方が重要になってきます。  
また、先ほどのmap関数に対してもこの高階関数で処理を行うこともできます。  
<br />
興味がある人は調べてみてください。

### store

middlewareAPI と呼ばれるもので、**createStore で作成された store とは異なります。**  
実態は getState と dispatch という関数を持っているオブジェクトです。

|   API    |                   説明                    |
| :------: | :---------------------------------------: |
| getState | Redux 内で管理されている state を取得する |
| dispatch |         新しく Action を発行する          |

#### getState

middleware 内で store の値にアクセスしたい場合に getState という関数を実行することで、
store オブジェクトを取得することができます。  
取得できるオブジェクトは combineReducers で登録されているキー値と同等です。

```js
// addItem.js
const { items, textValue } = store.getState().addItemState;
// itemList.js
const { items } = store.getState().addItemState;
```

コード中では、reducers/addItem.js 内で作成した state に対してアクセスをしています。  
combineReducers では addItemState という名前で reducer が登録されているので、
この形でアクセスをすることができています。

#### dispatch

middleware では他の Action を発行することができ、
この時使用するのがこの dispatch という関数になります。
dispatch は第 1 引数に ActionCreator を渡すことができ、container の dispatch と同じ要領で使えます。

```js
// addItem.js
import { textChange as textChangeActionCreator } from '../actions';

store.dispatch(textChangeActionCreator(''));
```

コード中では、textChange Action に空文字を payload に指定しています。
こうすることで、追加ボタンで家電を登録した際に入力フォームを初期化するという機能を実装することができます。

後述する next も同じようなことをしていますが、役割が違うので注意してください。

### next

指定された Action を次の Middleware もしくは reducer に渡す関数になります。  
Action が渡される順番は、

1. Middleware : middlewares/index.js で applyMiddleware した順に渡される
1. reducer : reducers/index.js で combineReducers した順に渡される

という流れになっています。
reducer に Action を渡すためには、必ず実行する必要があるので注意してください。

このように、middleware の next を使うことによって
middleware から middleware を呼び出して、更に middleware を呼び出し・・・最終的に reducer に流す  
という middleware のチェインを繋ぐことができます。

#### store.dispatch と next の違い

dispatch も next も第 1 引数に Action を渡して実行して、処理を流すという点では同じです。  
この 2 つの関数は役割が似ていますが、起点となるスタートに違いがあります。  
また、next 先が middleware ではなく reducer であった場合、next 実行後に state が更新されるという違いもあります。

dispatch は新しい Action を発行して Middleware の頭から Action に一致するものを探すのに対して、
next は Action を次の middleware もしくは reducer に渡しています。

```
① 実行フロー
// dispatch
Action1 発行 => middleware => store.dispatch(Action2) => redcuer(Action1)
                                                       Action2 発行 => redcuer(Action2)

// next
Action1 発行 => middleware1 => next(Action1) => reducer
                                                    => next(Action2) => middleware2 => redcuer

② データフロー
console.log(store.getState().hoge); // { key1: 'foo', key2: 'bar' };
next(updateKey1('foobar'));
console.log(store.getState().hoge); // { key1: 'foobar', key2: 'bar' };
```

このように実行フローとデータフローに store.dispatch と next には大きな違いがあります。  
なお、実行フローに関しては Action, Middleware, reducer の構成によっては
store.dispatch だけであるいは next だけで同じ振舞いをさせることは可能です。

### action

middleware が処理を実行する Action です。  
reducer と同様に switch 文で Action の振り分けを行っています。
もちろん、if 文で書いてもかまいません。

## Middleware の合成

次に 2 つある Middleware を合成します。reducer の合成の時と同じ要領で行います。
middlewares/index.js を変更します。

```js
// middlewares/index.js
import { applyMiddleware } from 'redux';
import changePowerState from './itemList';
import addItem from './textField';

const middlewares = applyMiddleware(addItem, changePowerState);

export default middlewares;
```

applyMiddleware に Middleware を渡すことで、1 つの Middleware にまとめることができます。

## Middleware の登録

App.js を変更し Middleware を登録します。  
また、ItemList を読み込めるように Provider に ItemList を追加します。

```jsx
import React from 'react';
import ReactDom from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import reducers from './reducers';
import middlewares from './middlewares';
import { ItemList, AddItem } from './containers';

const store = createStore(reducers, middlewares);

// DOM出力
ReactDom.render(
    <Provider store={store}>
        <>
            <AddItem />
            <ItemList />
        </>
    </Provider>,
    document.getElementById('content'),
);
```

Middleware の登録は、createStore 関数の第二引数で行います。

## Action の変更

家電の電源が押下された際に、どの家電かを示す情報を持たせられるように
changePowerState に payload を受け取れるようにします。  
main.js のファイル名を itemList.js に変更し、下記のコードに変更します。

```js
// actions/itemList.js
import { CHANGE_POWER_STATE } from './actionTypes';

export const changePowerState = payload => ({
    type: CHANGE_POWER_STATE,
    payload,
});
```

## Component の変更

### AddItemn の変更

登録された家電と電源状態を表示できるように変更します。  
まずは、Action と同様にファイルとコンポーネント名を ItemList に変更します。

```jsx
// components/ItemList.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import styled from 'styled-components';
import Switch from './Switch';
import DisplayState from './DisplayState';

class ItemList extends Component {
    static defaultProps = {
        changePowerStateAction() {},
        items: [],
    };

    render() {
        const { items, changePowerStateAction } = this.props;
        return (
            <div>
                {items.map((item, index) => (
                    <List key={item.id}>
                        <Item>{item.name}</Item>
                        <DisplayState power={item.power} />
                        <Switch onClick={changePowerStateAction(index)} />
                    </List>
                ))}
            </div>
        );
    }
}
ItemList.propTypes = {
    changePowerStateAction: PropTypes.func,
    items: PropTypes.array,
};

const List = styled.div`
    width: 100%;
    display: flex;
    color: rgb(100, 100, 100);
    padding-bottom: 1rem;
    margin-bottom: 2rem;
    border-bottom: 1px solid rgb(100, 100, 100);
`;
const Item = styled.div`
    font-size: 3rem;
    width: 30%;
`;

export default ItemList;
```

家電が入っている items を props として受け取れるようにし、
リスト表示できるように簡単ですが、スタイルを適用しました。

```jsx
<Switch onClick={changePowerStateAction(index)} />
```

家電の電源を切り替える Action ですが、このように map 関数の index を利用して  
家電の index と middleware で利用する action.payload の index をここで合わせています。

### styled-components について

スタイル部分には、styled-components という CSS-in-JS のモジュールを Web 開発基盤では採用しています。  
styled-components は従来の CSS と同等の書き方でスタイリングを記述することができ、
定義した名前で React Component として扱うことができます。

また、hover や MediaQuery など擬似要素や擬似クラスも表現することができます。

### DisplayState の変更

見た目を整えるために、スタイリングに手を加えます。

```jsx
render() {
    return <div style={{ width: '45px' }}>{this.props.power ? 'ON' : 'OFF'}</div>;
}
```

div エレメントに対して

`style={{ width: '45px' }}`

を追加します。

### AddItem の変更

フォームの入力内容の表示を ItemList コンポーネントの方に組み込んだので render メソッド内の

```jsx
{
    this.props.items.join('/');
}
```

が不要になったので、削除します。

## container の変更

Main.js を ItemList.js にリネームして、changePowerStateAction を変更します。

```jsx
// containers/ItemList.js
import { connect } from 'react-redux';
import { ItemList } from '../components';
import { changePowerState } from '../actions';

const mapStateToProps = ({ addItemState: { items } }) => ({
    items,
});

const mapDispatchToProps = dispatch => ({
    changePowerStateAction(index) {
        return () => {
            dispatch(changePowerState(index));
        };
    },
});

export default connect(
    mapStateToProps,
    mapDispatchToProps,
)(ItemList);
```

変更点は 2 点です。

-   changePowerStateAction が index を受け取れるようにした
-   return 文を追加して、関数を返す関数（高階関数）に変更した

高階関数にした理由ですが、ItemList.js では changePowetStateAction は下記のように

```jsx
<Switch onClick={changePowerStateAction(index)} />
```

先に index を渡しています。  
もし、ただの関数のままだと、index を受け取った段階で Action が発火してしまいます。  
なので、Switch ボタンが押下された時に渡されるコールバックを受け取れる状態にするために
高階関数に変更しています。

## reducer の変更

middleware 内で全て処理をした上で reducer に渡されるようにしたので、
addItem を payload をただ受け取るのみに変更します。

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
                items: payload,
            };
        }
        default: {
            return state;
        }
    }
};
```

以上でこの章での作業は終了です。  
なお、reducer 内には main.js が残ったままですが、こちらは使わなくなりましたので削除しても動作には問題ありません。
