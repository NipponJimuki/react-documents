## アプリケーションの実装

前のページでは TypeScript の基本について解説しました。
ここからは、React × Redux × TypeScript でアプリケーションを実装していきます。  
作成するアプリケーションは、<a href='../HowToRedux/HowToRedux1'>HowToRedux</a>と同じものです。  
まったく同じものを TypeScript を使って実装します。  
このページで作成するものは<a href='../HowToRedux/HowToRedux2'>HowToRedux2</a>となります。

## ソースコードの入手

tortoiseGit を使用して、テンプレートをリポジトリからクローンしてローカルに作成します。

クローンをするために、リモートリポジトリの URL が必要なのであらかじめコピーしておきます。

```
https://github.com/NipponJimuki/HowToTypeScript.git
```

<br />
作業を開始するディレクトリ上で右クリックして Git Clone を押します。
チュートリアルを始めるに当たってこちらのリポジトリを作業フォルダにクローンして開始してください。  
master ブランチがこのチュートリアルで使用するテンプレートになっています。

## ソースコードの確認

Git Clone してきたソースコードを確認します。  
src 直下の App.tsx を開きます。

```tsx
// App.tsx
import React from 'react';
import ReactDom from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';

const store = createStore(() => {});

// DOM出力
ReactDom.render(
    <Provider store={store}>
        <div>Text</div>
    </Provider>,
    document.getElementById('content'),
);
```

こちらが TypeScript 版の React ソースコードになります。  
ソースコード自体は変わりませんが、拡張子が js => tsx に変わりました。
拡張子を tsx にすることで、コンパイラー(webpack)が TypeScript で書かれた React のファイルであることを認識します。  
tsx ファイルは、コード上で jsx を扱えるファイルであり、React のコードを書く際は必ず tsx にする必要があります。

# ItemList を作成する

ボタンを押すと、ON / OFF の表示を切り替えるコンポーネントを作成します。  
HowToRedux2 では Main としていましたが、このツアーでは最初から ItemList で作成します。

# Component の作成

## ルートとなる ItemList

components フォルダに ItemList.tsx を作成し、下記コードを作成します。

```tsx
// components/ItemList.tsx
import React from 'react';
import Switch from './Switch';
import DisplayState from './DisplayState';

type Props = {
    power: boolean;
    changePowerStateAction: () => void;
};

class ItemList extends React.Component<Props> {
    static defaultProps: Pick<Props, 'power' | 'changePowerStateAction'> = {
        power: false,
        changePowerStateAction() {},
    };
    render() {
        const { power, changePowerStateAction } = this.props;
        return (
            <div>
                <Switch onClick={changePowerStateAction} />
                <DisplayState power={power} />
            </div>
        );
    }
}

export default ItemList;
```

js ファイルと比較して変わったものは以下の 3 点です。

-   type alias で Props というデータ型を宣言
-   defaultProps に型注釈をつけた
-   PropTypes の削除

TypeScript でコンポーネントを作成する場合、基本的に PropTypes をつけません。

React Component の型定義は

`React.Component<Props>`

このように行い、Props を型引数として渡します。  
これが TypeScript で作る React Component の雛形となります。

### Props の型定義

React の Props はオブジェクトのキーバリューで渡ってくるので、下記のように

```ts
type Props = {
    power: boolean;
    changePowerStateAction: () => void;
};
```

type alias を使って型定義を行います。
なお、interface で型定義を行なっても動作します。  
関数の型定義には

`changePowerStateAction: () => void;`

のように行い、() => void でボタンを押すことでアクションが発火される関数という型定義を行います。

### defaultProps の型定義

defaultProps に型注釈をしたい場合は、Pick を使う方法と Partial を使う方法の 2 通りの実装方法があります。

```ts
static defaultProps: Pick<Props, 'power' | 'changePowerStateAction'> = {
    power: false,
    changePowerStateAction: () => {},
};
```

ここでは Pick を使って型注釈を行なっています。  
Pick とはその名の通り、型の集合体からある特定の型のみを抽出するという機能になります。  
受け取る props 全てに型注釈を行なっていますので、実は Pick を使わなくてもいいのですが、今回は紹介するために Pick を使っています。

通常、type alias で指定した props はコンポーネント宣言時に省略することができません（任意のプロパティに指定すれば省略可）。
TypeScript v3.0.0 から defaultProps がサポートされ、
defaultProps に指定した props はコンポーネント宣言時に省略してもエラーにならないようになっています。

#### React Component の型定義

React × Redux のアプリケーションなので Props のみを型定義していますが、
React.Component は 3 つの型引数を受け取ることができます。

| 型引数 | 型  |      説明      |
| :----: | :-: | :------------: |
| Props  | {}  | Props の型定義 |
| State  | {}  | State の型定義 |
|   SS   | any |   その他の型   |

第 3 引数の SS ですが、定義元の@types/react/index.d.ts の Component を見てもどこにも使われていません。  
おそらく、サードパーティや HOC を利用したコンポーネントに定義する型だと思われます。

## DumbComponent の作成

ItemList が呼び出している Switch.tsx と DisplayState.tsx をそれぞれ作成します。

```tsx
// components/Schitw.tsx;
import React from 'react';

type Props = {
    onClick: () => void;
};

const Switch: React.SFC<Props> = ({ onClick }) => <button onClick={onClick}>スイッチ</button>;

export default Switch;
```

```tsx
// components/DisplayState.tsx
import React from 'react';

type Props = {
    power: boolean;
};

const DisplayState: React.SFC<Props> = ({ power = false }) => <div>{power ? 'ON' : 'OFF'}</div>;

export default DisplayState;
```

親コンポーネントで作成した Props の型定義を子のコンポーネントでもそれぞれ行うようにします。  
stateless functional component(以下 SFC)の型定義は

`React.SFC<Props>`

このように SFC という型定義名を使います。  
SFC は Component と違い、受け取れる型引数は 1 つのみとなっていて、Props のみ型引数として受け取れるようになっています。

## ItemList を export する

components 配下に index.ts を作成し、ItemList コンポーネントのメインとなる ItemList.tsx を export します。

```ts
// components/index.ts
export { default as ItemList } from './ItemList';
```

注意点として、コンポーネントは tsx という拡張子ですが、こちらは ts という拡張子になっています。  
JSX を扱わないので、普通の ts ファイルになっています。

# Container の作成

作成したコンポーネントを Redux と接続します。
containers 配下に ItemList.tsx を作成し、下記コードを作成します。

```tsx
// containers/ItemList.tsx
import { connect } from 'react-redux';
import { Dispatch } from 'redux';
import { ItemList } from '../components';
import { changePowerState } from '../actions';
import { Store } from '../reducers';

const mapStateToProps = ({ powerState }: Store) => ({
    ...powerState,
});

const mapDispatchToProps = (dispatch: Dispatch) => ({
    changePowerStateAction() {
        dispatch(changePowerState());
    },
});

export default connect(
    mapStateToProps,
    mapDispatchToProps,
)(ItemList);
```

続いて、コンポーネントと同様に export するための index.ts を作成します。

```ts
// containers/index.tsx
export { default as ItemList } from './ImteList';
```

Container でつけるべき型は

-   mapStateToProps
-   mapDispatchToProps

この 2 つの関数に対してです。

## mapStateToProps の型定義

mapStateToProps は Redux の store の中から必要な state のみを props として渡す関数となります。  
なので

<br />
mapStateToPropsの型 = storeの型

<br />
と同じでなければなりません。  
そして、作成されるContainerの数は1つとは限りません。  
つまり、mapStateToPropsに渡す型は再利用できるように
別ファイルに型定義したものを使った方が無駄な手間をなくすことができます。
ここでは、アプリケーションのルートとなるAppにStoreという名前で型を作成してimportして使っています。

## mapDispatchToProps の型定義

mapDispatchToProps は Redux の action を props としてコンポーネントに渡します。

`Dispatch`

という型定義を Redux が export していますので、こちらを使います。  
Dispatch という型定義は type プロパティを持つオブジェクトであれば何でも許容する型となっています。

# Action の作成

ボタンを押下した際に発火されるアクションを作成します。  
まずは、アクションタイプファイルの作成をします。  
actions 配下に actionType.ts を作成し、下記コードを作成します。

```ts
// sctions/actionType.ts
export const CHANGE_POWER_STATE = 'CHANGE_POWER_STATE';
```

今後、ここに全てのアクションタイプを定義していきます。  
続いて、ItemList が受け取る actionCreator を作成します。

```ts
// actions/itemList.ts
import { CHANGE_POWER_STATE } from './actionType';

export const changePowerState = () => ({
    type: CHANGE_POWER_STATE as typeof CHANGE_POWER_STATE,
});
```

こちらも、index.ts を作成して export します。

```ts
// actions/index.ts
export * from './itemList';
```

## actionCreator の型定義

基本的に普通の JavaScript と変わりはありませんが、一点だけ異なる点があります。

`type: CHANGE_POWER_STATE as typeof CHANGE_POWER_STATE`

type プロパティに渡した定数の後に as typeof を使ってキャストを行なっています。  
こうすることで、type プロパティをただの string ではなく CHANGE_POWER_STATE という型で定義します。

# reducer の作成

ItemList 用の state を作成します。  
ここで先ほど作成した action の型定義を使います。  
reducer では TypeScript 特有の機能を複数組み合わせて使います。

## ItemList の state を作成

reducers 配下に itemList.ts を作成し、下記コードを作成します。

```ts
// reducers/itemList.ts
import { Reducer } from 'redux';
import { CHANGE_POWER_STATE } from '../actions/actionType';
import { changePowerState } from '../actions';

type Action = ReturnType<typeof changePowerState>;
export type PowerState = {
    power: boolean;
};
const initialState = {
    power: false,
};

const powerState: Reducer<PowerState, Action> = (state = initialState, action) => {
    switch (action.type) {
        case CHANGE_POWER_STATE:
            return {
                ...state,
                power: !state.power,
            };
        default:
            return state;
    }
};

export default powerState;
```

こちらも export します。

既存のコードと比べると、変更箇所は多いです。

-   Conditional Types（ReturnType）を使って Action を型定義
-   State の型定義をし、export する
-   作成した型を使って Reducer の型定義
-   Tagged Union Types で Action の絞り込み

ということを行なっています。  
reducer では Conditional Types と Tagged Union Types を使って型安全にすることができるのですが、
受け取る Action が 1 つだと、効果を発揮することができません。  
なので、説明は後のツアーで行います。  
今は、reducer は TypeScript だとこのように記述するぐらいの認識で大丈夫です。

## Store の型定義

続いて、reducer を export します。  
ただし、他のファイルと違いここで reducer を作成し更に Store の型定義も行います。

```ts
// reducers/index.ts
import { combineReducers } from 'redux';
import powerState, { PowerState } from './itemList';

export type Store = {
    powerState: PowerState;
};

const reducers = combineReducers({
    powerState,
});

export default reducers;
```

Store はここで使うのではなく、作成した Container と後に作成する Middleware で使います。  
以上で、Redux 部の実装は終わりです。

# App.ts の変更

App.ts に作成した Reducer を流し込みます。

```tsx
// App.tsx
import React from 'react';
import ReactDom from 'react-dom';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import redcuers from './reducers';
import { ItemList } from './containers';

const store = createStore(redcuers);

ReactDom.render(
    <Provider store={store}>
        <ItemList />
    </Provider>,
    document.getElementById('content'),
);
```

作業は以上となります。  
JavaScript と違い、型定義を行うために全体的に記述量が大きくなっています。  
また、React / Redux のテンプレートと異なっている箇所もあります。
逆を言えば、少しの記述量で型安全にかつ型の恩恵にあやかりつつアプリケーションを作成することができます。  
大規模になればなるほど、この恩恵は大きくなります。
