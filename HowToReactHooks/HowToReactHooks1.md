## ツアーについて

こちらは上級者用のツアーになります。  
プログラミングガイド（基盤）まで含む、全てのツアーを終えており、
**ESNext, React, Redux, TypeScript についての一通りの知識がある前提で行います。**  
**なお、ツアー内で ESNext、TypeScript の構文の解説も行いません。**

HowToRedux で実装したものを React Hooks の機能のみを使って実装しなおします。  
実装は全て TypeScript / React です。  
また、いくつか機能の追加も行います。

2019 年 4 月 2 日現在、React の最新バージョンは 16.8.6  で、16.8.0 以上であれば React Hooks を使うことができます。

なお、言及してるものの多くは筆者の憶測が多分に含まれておりますので、参考程度の意見として見てください。

## React Hooks

React v16.8.0 より React Hooks（以下、Hooks）という機能が新しく追加されます。

今までの React の実装はクラスベースの Statefull Class Component と関数ベースの Stateless Functional Component（以下、SFC）の 2 つがありました。
Class Component では state（状態）が扱えるのに対して、SFC では state を扱うことができません。

Hooks の機能を使うことで、関数ベースでありながら state やライフサイクルメソッドを扱うことができるようになります。

Hooks では useState や useReducer など、React ユーザにとって見覚えのある言葉がいくつかでてきます。  
また、recompose という Higher Order Components（以下、HOC）ベースのライブラリーを使ったことがあるユーザにとっても
見覚えのある書き方がよくでてきます。

なお、componentDidCatch 相当の振る舞いやテストの仕方など、バグや未対応の実装も少なくないです。  
これから考えていくとのことです。

### Hooks の登場によって変わったもの変わりそうなもの

今までデファクトスタンダードだったものが変わり、非推奨となる技術まででています。
以下が、Hooks の登場によって変わったものと変わりそうなものです。

-   変わったもの
    -   recompose : メンテナンスが打ち切られた
    -   SFC : Function Component に名前が改められた（SFC という言葉はなくなる）
    -   Redux : react-redux が Hooks との接続を行う対応を開始した
-   変わりそうなもの
    -   Class Component : いずれ非推奨の書き方になると予想される
    -   HOC : コードの見通しが悪くなる機能と公式が言及
    -   rednerProp : HOC の煽りを受け、どうなるかがわからない
    -   各種ライブラリー : Hooks に対応されないものは淘汰されていく

HOC や renderProp は、非推奨の技術とまでは言われていませんが、これらは
Function Component で全て代替できるようになるため、使われなくなっていく技術になると思われます。

また、React の各種サードパーティ（Formik や styled-components）も Hooks に対応してるか否かが
ライブラリー選定の大きな分かれ目となっていくことが予想されます。

#### Class Component と SFC について

Facebook は Class Component を断ち切るつもりはないと明言していますが、
これからは、全てのコンポーネントが Function Component で作成されていくような流れにやがて変わっていきます。

SFC については、言葉そのものが無くなります。  
最新の@types/react の実装では、FucntionComponent に既に改められています。
互換性を保つために React.SFC という型定義は存在しますが、非推奨の型定義となっています。

```ts
// node_modules/@types/react/index.d.ts

/**
 * @deprecated as of recent React versions, function components can no
 * longer be considered 'stateless'. Please use `FunctionComponent` instead.
 *
 * @see [React Hooks](https://reactjs.org/docs/hooks-intro.html)
 */
type SFC<P = {}> = FunctionComponent<P>;

/**
 * @deprecated as of recent React versions, function components can no
 * longer be considered 'stateless'. Please use `FunctionComponent` instead.
 *
 * @see [React Hooks](https://reactjs.org/docs/hooks-intro.html)
 */
type StatelessComponent<P = {}> = FunctionComponent<P>;
```

### Redux について

ContextAPI や Suspense、Hooks を駆使することで、Redux 無しのアプリケーション構築がいよいよ現実味を帯びてきた感はあります。  
ただ、redux-logger や redux-saga など現状の React では取って替わることのできない多くの機能があるのもまた事実です。

着実に現実味は帯びてきていますが、まだまだ現実的ではないと思われます。

Redux はピュアな関数群であるため、直接的には影響はありません。  
現在、React と Redux の接続を担う react-redux 側が Hooks との対応を行っています。

react-redux v7.0.0-beta.0 で React との接続を行う ReactReduxContext が外だしされるようになっており、
こちらを使うと Redux と組み合わせることは可能です。  
ただし、React が v16.8.4 以降のバージョンであることが必須となっております。

react-redux 側は、すぐに Hooks に対応するつもりはないと言及しており、議論を重ねてどう対応していくか考えていく状態です。  
現状の実装はまだまだ実験的なものであると思われ、プロダクション環境で使うには十分な検討が必要です。

## React Hooks で追加された API

現在、Hooks で使える API は 10 こあります。  
全ての API は use から始まる名称となります。  
また、Custom Hooks を作ることもでき、その場合も use から始まる名称であるべきと紹介されています。

#### HooksAPI のカテゴライズ

HooksAPI は Basic Hooks と Additional Hooks の 2 つに分かれており、カテゴライズは以下の通りになっています。

-   Basic Hooks
    -   useState
    -   useEffect
    -   useContext
-   Addtional Hooks
    -   useReducer
    -   useCallback
    -   useMemo
    -   useRef
    -   useDebugValue
    -   useImperativeHandle
    -   useLayoutEffect

### useState

その名の通り、コンポーネントが持つ state を定義する API になります。  
今まではオブジェクトで state を持つようにしていましたが、Hooks では単一の状態とそれを更新するための関数を 1 セットに state を定義します。

```js
// Class Component でのstate宣言
this.state = {
    text: '',
    total:, 0,
    disabled: false,
};

// Function Component でのstate宣言
const [text, changeText] = useState('');
const [count, setCount] = useState(0);
const [disabled, clickableHandler] = useState(false);
```

useState の第 1 引数に初期値となる値を渡してあげることで、初回レンダリング時にその値がコンポーネントに渡されるようになります。  
useState からの返却値取得は分割代入を使い、第 1 返却値に state、第 2 返却値に state を更新するための関数が入っています。

#### state を更新する関数

今までの state 更新には、直接値を与える方法とコールバックで更新する方法がありました。  
Hooks での state 更新も同様のことを行えます。

```jsx
function TodoComp() {
    const [text, changeText] = useState('');
    const [todos, addTodo] = useState(0);

    return (
        <>
            <input type="text" onChange={e => changeText(e.target.value)}
            <button onClick={() => addTodo(prevTodos => [...prevTodos, text])}>追加</button>
            {todos.map(todo => <div>{todo}</div>)}
        </>
    )
}
```

### useEffect

イベントリスナーの追加や DOM のアタッチなどの副作用を記述する時はこの API を使います。  
componentDidMount, componentDidUpdate, componentWillUnMount, getSnapshotBeforeUpdate 相当の役割を担います。  
useEffect は 2 つの引数を取ることができ、第 1 引数に処理を、第 2 引数にメモライズキーを渡すことができます。  
この第 2 引数と内部の処理によっては getDerivedStateFromProps 相当の役割を果たすこともできます。

第 2 引数のメモライズキーとは配列で、配列の中に渡した値に変更があった時だけ useEffect を実行させるトリガーとなります。  
この引数によって振る舞いが変わる性質を利用し、異なるライフサイクルメソッド相当の動きを行います。

-   第 2 引数に空の配列を渡す => componentDidMount, componentWillUnMount
-   第 2 引数に配列を渡す => componentDidUpdate, getSnapshotBeforeUpdate

useEffect は React のライフサイクルメソッドと違い、DOM 更新処理後に非同期で呼ばれます。  
同期的に呼ばれることを保証したい場合は後述する useLayoutEffect を使います。

#### componentDidMount, componentWillUnMount の実装

```js
const [scroll, setScroll] = useState(window.innerHeight);

useEffect(() => {
    // ここはcomponentDidMount相当の処理
    const onScroll = () => setScroll(window.scrollY);
    window.addEventListener('scroll', onScroll);
    return () => {
        // この中はcomponentWillUnMount相当の処理
        window.removeEventListener('scroll', onScroll);
    };
}, []);
```

これは、現在のスクロール位置を取得し、スクロールする度に scroll 値を更新する処理を行なっています。  
空の配列を指定することで、コンポーネントの状態の有無に関与されなくなるので、一度しか実行されないことが保証されます。

#### componentDidUpdate の実装

```js
const [fake, setFake] = useState(false);
const [count, setCount] = useState(0);

useEffect(() => {
    console.log(`current value = ${count}`);
    if (count === 10) {
        console.log('maximum count');
        setCount(0);
        console.log('reset complete!!');
    }
}, [count]);
```

この useEffect は count 値を監視しており、count 値に変更があった場合にのみ実行されます。
10 に達したらリセットするような処理を行います。  
この際、fake が更新されても useEffect が実行されることはありません。

#### getSnapshotBeforeUpdate の実装

```jsx
function RectComponent({ location }) {
    const divEl = useRef(null);
    const [rect, setRect] = useState({});

    useEffect(() => {
        if (divEl) {
            const clientRect = divEl.getBoundingClientRect();
            setRect(clientRect);
        }
    }, [location.pathname]);

    /* 以下省略 */
}
```

この useEffect は props.location を監視しており、ロケーション情報に変更があった場合にクライアント座標をセットし直しています。  
useEffect は DOM 更新処理後に呼ばれ、DOM にアクセスすることもできます。  
このように DOM の値を参照をする場合は getSnapshotBeforeUpdate 相当の実装になります。

## useContext

Hooks で ContextAPI を使う場合に使用する API になります。  
ツアー内で ContextAPI を使った実装はしていませんが、Redux で connect していたコンポーネントを useContext を使って同等の操作を行うことができます。

使用シーンとしては、後述する useReducer を使って状態と更新をまとめあげたものを、useContext を使って props として受け取るということができます。  
イベントのトリガーとなる dispatch 関数のバケツリレーを回避することなどができます。

```jsx
export const CounterContext = React.createContext(null);

const Provider = ({ children }) => {
    const [state, dispatch] = useReducer(rootReducer, undefined, { type: 'INIT' });

    return (
        <CounterContext.Provider value={{ state, dispatch }}>{children}</CounterContext.Provider>
    );
};

const App = () => (
    <Provider>
        <Counter />
        <AdjustButtons />
    </Provider>
);
```

Provider の作成は今まで通り、ContextAPI で作成したコンポーネントに対して、管理したい値を value に渡すことで
子コンポーネントで取り出すことができるようになります。  
Provider の子コンポーネントである Counter と AdjustButtons で Context を受け取る場合は

```jsx
import { CounterContext } from './Root';

const Counter = () => {
    const { state } = useContext(CounterContext);

    return (
        <div>{state.count}</div>
    );
}

const AdjustButtons = () => {
    const { dispatch } = useContext(CounterContext);

    return (
        <button onClick={() => dispatch({ type: 'INCREMENT' })}>＋</button>
        <button onClick={() => dispatch({ type: 'DECREMENT' })}>ー</button>
    );
}
```

このように useContext に ContextAPI で作成したコンポーネントを渡すことで値を取り出すことができます。

### useReducer

useState の代替 API で、状態と更新の全てを useReducer に任せることができるようになります。  
Redux で扱う reducer と同等の状態管理にすることができます。

```js
const initialState = {
    text: '',
    count: 0,
    disabled: false,
};

const reducer = (state: State, action: Action) => {
    switch (action.type) {
        case 'RESET':
            return initialState;
        case 'CHANGE_TEXT':
            return { ...state, text: action.payload };
        case 'SET_COUNT':
            return { ...state, count: action.payload };
        case 'CLICKABLE_HANDLER':
            return { ...state, disabled: !state.disabled };
    }
};

function TodoComp() {
    const [state, dispatch] = useReducer(reducer, initialState);

    /* 以下省略 */
}
```

useReducer は 3 つの引数を受け取ることができ

`const [state, dispatch] = useReducer(reducer, initialState, { type: 'reset' });`

とすることで、初回レンダリング時に redcuer 内の RESET ケースの値が返却されるようになります。  
useReducer が返却する dispatch という変数ですが、これは action を dispatch する関数となっています。

`<button onClick={() => dispatch({ type: 'SET_COUNT', payload: 0 })}>リセット</button>`

のように使います。  
mapDispatchToProps で行われていた dispatch(action(value))と同じような使い方となっています。  
action の type プロパティ（と payload）を渡して、状態の更新ケースを分けて更新させる流れとなります。

### useCallback

メモ化されたコールバック関数を返す API です。  
useEffect 同様、第 2 引数にメモライズキーを渡すことで、関心ごとを分離できます。  
メモライズキーに変更があった場合のみに関数を再作成して渡すことができるので、onClick や onChange に props 経由で渡す場合のレンダリングを抑制することができます。多くの場合、React.memo と併用して使うことが多いです。

```jsx
function CustomButton({ label, onClick }) {
    return <button onClick={onClick}>{label}</button>;
});
React.memo(CustomButton);

function ResetButton({ label = 'リセット' }) {
    const onClick = useCallback(() => window.alert('done!!'), []);

    return <CustomButton label={label} onClick={onClick} />;
}
```

このコードでは、親となる ResetButton で onClick を定義しています。  
この時、useCallback の第 2 引数には空の配列が指定されるので一度定義された onClick が再作成されることはありません。  
子となる CustomButton は React.memo でメモライズ化しているので、label に変更があった場合のみ再レンダリングされるようになります。

### useMemo

メモ化された値を返す API です。
こちらも第 2 引数にメモライズキーを渡すことで、関心ごとを分離できます。
使い方としては、useCallback と同じように不要なレンダリングを避ける目的で使うことが多いです。

```jsx
const [count, setCount] = useState(0);
const [text, changeText] = useState('');
const isReset = useMemo(() => count === 10, [count]);

return (
    <>
        <button onClick={() => setCount(prevCount => prevCount + 1)} disabled={isReset}>
            加算
        </button>
        {isReset && <button onClick={() => setCount(0)}>リセット</button>}
        <input type="text" onChange={e => changeText(e.target.value)} />
        <div>currentInputValue: {text}</div>
    </>
);
```

ボタンによるカウントとテキストを入力できるコンポーネントで、isReset の値は count 値に変更があった場合にのみ比較を行い
値の変更を監視するということができるようになります。  
ボタンは 10 までカウントできますが、それ以上カウントできなくなり、リセットボタンの押下をユーザに促せることができます。

### useRef

Hooks で ref を使う場合に使用する API です。  
createRef で作成していた ref が useRef に変わっています。

```jsx
const inputEl = useRef(null);
useEffect(() => {
    if (inputEl) inputEl.current.focus();
}, []);

return <input type="text" ref={inputEl} />;
```

初回レンダリング時に input エレメントにフォーカスするような場面で使ったりします。
基本的な利用用途は今までの ref と同じになります。

### useDebugValue

16.8.0 より新しく追加された API になります（Hooks は 16.7.0 のアルファ版から使えるようになっておりました）。  
その名の通り、デバッグに使う Hooks です。  
主な使用用途は、共有ライブラリ等で独自のフックシステムを構築しているカスタムフックの現在値チェックなどに使います。

```jsx
function useCustomHooks() {
    const [text, setText] = useState('');
    const [todo, setTodo] = useState([]);

    useDebugValue(`現在のstateはtext: ${text}、todo: ${JSON.stringify(todo)}です`);

    const addTodo = useCallback(
        value => {
            const newTodo = [...todo];
            newTodo.push({ id: performance.now(), value });
            setTodo(newTodo);
        },
        [todo],
    );

    return {
        text,
        setText,
        item,
        addTodo,
    };
}

function TodoList() {
    const { text, setText, item, addTodo } = useCustomHooks();

    return (
        <React.Fragment>
            <input type="text" onChange={e => setText(e.target.value)} />
            <button onClick={() => addTodo(text)}>追加</button>
            {item.map(key => (
                <p key={key.id}>{key.value}</p>
            ))}
        </React.Fragment>
    );
}
```

この簡易的な TodoList コンポーネントは、
state やメモライズ化した関数の作成などを useCustomHooks というカスタムフックの中で行なっています。

```js
useDebugValue(`現在の state は text: ${text}、todo: ${JSON.stringify(todo)}です`);
```

ここが useDebugValue の処理になり、text と todo の値を出力しています。  
React の開発者ツールの中で確認することができます（コンソールには出力されないので気をつけてください）。

<img width="100%" src='./src/markDowns/imgs/HowToReactHooks/useDebugValue.png'>

#### データフォーマットの遅延処理

useDebugValue は引数を 2 つ受け取ることができ、第一引数に指定された値を元に第二引数でデータのフォーマットを行うことができます。  
上記の useCustomHooks というカスタムフックを例にとると、

```js
useDebugValue(todo, todo => JSON.stringify(todo));
```

このように todo リストの受け取りと JSON シリアライズのタイミングをずらすことができます。  
useDebugValue は todo に変更があった時のみデバッグを出力するようになります。  
useEffect や useCallbak の第 2 引数と同じような振る舞いを useDebugValue は第 1 引数と第 2 引数を使って実装します。

### useImperativeHandle

ref に対して特定の処理を紐づけることができる際に使用する API です。
また、forwardRef を使って ref 経由で親からアクセスさせる際に、ref のオブジェクトをカスタマイズさせるといったことも
この API を使えば行うことができます。

使用シーンはそんなに多くないと思います。

### useLayoutEffect

基本的には useEffect と同じです。
全ての DOM の更新処理が終わったタイミングで同期的に呼ばれる API になります。

componentDidUpdate と同じタイミングで呼ばれます。

### useMutationEffect

**React 16.8.0 がリリースされたタイミングで削除されました。**  
**もし使っている場合、useEffect か useLayoutEffect に置き換えてください。**
