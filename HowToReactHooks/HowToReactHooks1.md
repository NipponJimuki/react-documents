## ツアーについて

こちらは上級者用のツアーになります。  
プログラミングガイド（基盤）まで含む、全てのツアーを終えており、
**ESNext, React, Redux, TypeScript についての一通りの知識がある前提で行います。**  
**なお、ツアー内で ESNext、TypeScript の構文の解説も行いません。**

HowToRedux で実装したものを React Hooks の機能のみを使って実装しなおします。
また、いくつか機能の追加も行います。  
実装は全て React/ TypeScript です。  


2018 年 12 月 21 日現在、React の最新バージョンは 16.7.0  で、React Hooks が使用できるのは 16.7.0-alpha.2 です。  
16.7.0 にも React Hooks の API は追加されていますが、リリースノートには 「No, This Is Not The One With Hooks」と記載されております。  
ツアーではalpha.2のバージョンを使用します。  

**こちらで紹介する書き方は将来大きく変更される可能性があります。**  
また、言及してるものの多くは筆者の憶測が多分に含まれておりますので、参考程度の意見として見てください。

## React Hooks

React v16.7.0 より React Hooks（以下、Hooks）という機能が新しく追加されます。

今までの React の実装はクラスベースの Class Component と関数ベースの Stateless Functional Component（以下、SFC）の 2 つがありました。
Class Component では state（状態）が扱えるのに対して、SFC では state を扱うことができません。

Hooks の機能を使うことで、関数ベースでありながら state やライフサイクルメソッドを扱うことができるようになります。

Hooks では useState や useReducer など、React ユーザにとって見覚えのある言葉がいくつかでてきます。  
また、recompose という Higher Order Components（以下、HOC）ベースのライブラリーを使ったことがあるユーザにとっても
見覚えのある書き方がよくでてきます。

なお、componentDidMount の振る舞いやテストの仕方など、バグや未対応の実装も少なくないです。
これから考えていくとのことです。

### Hooks の登場によって変わったもの変わりそうなもの

今までデファクトスタンダードだったものが変わり、非推奨となる技術まででています。
以下が、Hooks の登場によって変わったものと変わりそうなものです。


- 変わったもの
  - recompose : メンテナンスが打ち切られた
  - SFC : Function Component に名前が改められた（SFCという言葉はなくなる）
  - Redux : react-reduxがHooksとの接続を行う対応を開始した
- 変わりそうなもの
  - Class Component : いずれ非推奨の書き方になると予想される
  - HOC : コードの見通しが悪くなる機能と公式が言及
  - rednerProps : HOC の煽りを受け、どうなるかがわからない
  - 各種ライブラリー : Hooks に対応されないものは淘汰されていく


HOCやrenderPropsは、非推奨の技術とまでは言われていませんが、これらは
Function Componentで全て代替できるようになるため、使われなくなっていく技術になると思われます。

また、Reactの各種サードパーティ（Formikやstyled-components）もHooksに対応してるか否かが
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

Reduxはピュアな関数群であるため、直接的には影響はありません。  
現在、ReactとReduxの接続を担うreact-redux側がHooksとの対応を行っています。  

react-redux v6 で React との接続を行う ReactReduxContext が外だしされるようになっており、
こちらを使うと alpha.2 でも Redux と組み合わせることは可能です。  
ただし、state の変更に過敏で、レンダリングの抑制など自前で実装しないといけないなどの問題があります。  

react-redux側は、すぐにHooksに対応するつもりはないと言及しており、議論を重ねてどう対応していくか考えていく状態です。  
現状の実装はまだまだ実験的なものであると思われ、将来的には`useRedux`のようなAPIが作成されるのではないかと予想しています。

## React Hooks で追加された API

現在、Hooks で使える API は 10 こあります。
全ての API は use から始まる名称となります。  
また、Custom Hooks を作ることもでき、その場合も use から始まる名称であるべきと紹介されています。

#### HooksAPI のカテゴライズ

HooksAPI は Basic Hooks と Additional Hooks の 2 つに分かれており、カテゴライズは以下の通りになっています。

- Basic Hooks
  - useState
  - useEffect
  - useCotenxt
- Addtional Hooks
  - useReducer
  - useCallback
  - useMemo
  - useRef
  - useImperativeMethods
  - useLayoutEffect
  - useMutationEffect


**※ useMutationEffect は公式ページには記載されておりませんが、HooksAPI の 1 つとしてあります。**
**基本的に非推奨の Hooks に指定されているので、公式ページには記載がないものと思われます。**

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
componentDidMount, componentDidUpdate, componentWillUnMount 相当の役割を担います。  
useEffect は 2 つの引数を取ることができ、第 1 引数に処理を、第 2 引数にメモライズキーを渡すことができます。

第2引数のメモライズキーとは配列で、配列の中に渡した値に変更があった時だけuseEffectを実行させるトリガーとなります。
この引数によって振る舞いが変わる性質を利用し、異なるライフサイクルメソッド相当の動きを行います。

- 第2引数に空の配列を渡す => componentDidMount, componentWillUnMount
- 第2引数に配列を渡す => componentDidUpdate

useEffectはReactのライフサイクルメソッドと違い、DOM更新処理後に非同期で呼ばれます。  
同期的に呼ばれることを保証したい場合は後述するuseLayoutEffect, useMutationEffect を使います。

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

useEffect(
    () => {
        console.log(`current value = ${count}`);
        if (count === 10) {
            console.log('maximum count');
            setCount(0);
            console.log('reset complete!!');
        }
    },
    [count],
);
```

この useEffect は count 値を監視しており、count 値に変更があった場合にのみ実行されます。
10 に達したらリセットするような処理を行います。  
この際、fake が更新されても useEffect が実行されることはありません。

### useContext

Hooks で createContext を使う場合に使用する API になります。  
ツアー内で Context を使った実装はしていませんが、Redux で connect していたコンポーネントを useContext を使って同等の操作を行うことができます。

使用シーンとしては、後述する useReducer を使って状態と更新をまとめあげたものを、useContext を使って props として受け取るということができます。  
ツアーでも使わないので、詳しい説明は省きます。

### useReducer

useState の代替 API で、状態と更新の全てを useReducer に任せることができるようになります。  
useState の Redux の reducer 版です。

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
<br>

```jsx
const CustomButton = React.memo(function({ label, onClick }) {
    return <button onClick={onClick}>{label}</button>;
});

function ResetButton({ label }) {
    const onClick = useCallback(() => window.alert('done!!'), []);

    return <CustomButton label={label || 'リセット'} onClick={onClick} />;
}
```

<br>
このコードでは、親となる ResetButton で onClick を定義しています。  
この時、useCallback の第 2 引数には空の配列が指定されるので一度定義された onClick が再作成されることはありません。  
子となる CustomButton は React.memo でメモライズ化しているので、label に変更があった場合のみ再レンダリングされるようになります。

### useMemo

メモ化された値を返す API です。
こちらも第 2 引数にメモライズキーを渡すことで、関心ごとを分離できます。
使い方としては、useCallback と同じように不要なレンダリングを避ける目的で使うことが多いです。
<br>

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

<br>
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

### useImperativeMethods

ref に対して特定の処理を紐づけることができる際に使用する API です。
また、forwardRef を使って ref 経由で親からアクセスさせる際に、ref のオブジェクトをカスタマイズさせるといったことも
この API を使えば行うことができます。

使用シーンはそんなに多くないと思います。

### useLayoutEffect

基本的には useEffect と同じです。
全ての DOM の更新処理が終わったタイミングで同期的に呼ばれる API になります。

componentDidUpdate と同じタイミングで呼ばれます。

### useMutationEffect

基本的には useEffect と同じです。
React が DOM を更新するのと同じタイミングで同期的に呼び出されます。

DOM の更新中に実行されるため、使い方によってはパフォーマンス悪影響を及ぼす可能性があります。

特に理由がない限り、使わないようにする非推奨の API に指定されています。
