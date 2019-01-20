## 家電管理アプリケーションを実装する

前回のツアーでは簡易的な Todo を作成しました。  
今回は HowToRedux4 で作成した家電管理アプリケーションを実装します。

追加機能として、登録できる家電は 5 つまでとし、5 つ登録された時点で登録できないようにしてリセットボタンを出現させます。

## Reducer を作成する

今回は useState ではなく、useReducer を使って状態管理を行なっていきます。  
Redux では、action と reducer を別々に作成しましたが、
ここでは同じファイルに全て定義します（従来のように別ファイルに管理しても大丈夫です）。

components 配下に reducer.ts を作成し、以下のコードを作成します。

```ts
// components/reducer.ts
// action定数
const ADD_ITEM = 'ADD_ITEM';
const RESET = 'RESET';
const CHANGE_POWER_STATE = 'CHANGE_POWER_STATE';

// action関数
export const addItem = (payload: string) => ({
    type: ADD_ITEM as typeof ADD_ITEM,
    payload,
});

export const reset = () => ({
    type: RESET as typeof RESET,
});

export const changePowerState = (payload: number) => ({
    type: CHANGE_POWER_STATE as typeof CHANGE_POWER_STATE,
    payload,
});

export interface ItemProps {
    id: string;
    name: string;
    power: boolean;
}

type State = {
    items: ItemProps[];
};
type Action = ReturnType<typeof addItem | typeof reset | typeof changePowerState>;

export const initialState = {
    items: [],
};

// main reducer
const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case ADD_ITEM: {
            return {
                items: [
                    ...state.items,
                    {
                        id: `item-index-${state.items.length}`,
                        name: action.payload,
                        power: false,
                    },
                ],
            };
        }
        case RESET: {
            return {
                items: [],
            };
        }
        case CHANGE_POWER_STATE: {
            const { items } = state;
            items[action.payload].power = !items[action.payload].power;

            return {
                items,
            };
        }

        default: {
            const _: never = action;
            return state;
        }
    }
};

export default reducer;
```

基本的に、Redux の action と reducer と同じ書き方になります。  
また、TypeScript の Tagged Union Types や never 型での振い落としも同様に行うことができます。  
ここで作成した action と役割は以下の通りです。

-   ADD_ITEM : 家電を追加するアクション
-   RESET : 家電を初期化する
-   CHANGE_POWER_STATE : 対象の家電の電源を切り替える

## ルートとなるコンポーネントの作成

家電を登録する AddItem コンポーネントと家電の一覧を表示する ItemList コンポーネントをまとめあげる App コンポーネントを作成します。  
components 配下に index.tsx を作成し、下記コードを作成します。

```tsx
// components/indes.tsx
import React, { useReducer, useMemo } from 'react';
import reducer, { initialState, addItem, reset, changePowerState } from './reducer';
import AddItem from './AddItem';
import ItemList from './ItemList';

function App() {
    const [{ items }, dispatch] = useReducer(reducer, initialState);
    const disabled = useMemo(() => items.length >= 5, [items]);
    const onAddItem = (item: string) => dispatch(addItem(item));
    const onReset = () => {
        if (disabled) dispatch(reset());
    };
    const onChangePower = (index: number) => () => dispatch(changePowerState(index));

    return (
        <>
            <AddItem onAdd={onAddItem} onReset={onReset} disabled={disabled} />
            <ItemList items={items} onClick={onChangePower} />
        </>
    );
}

export default App;
```

ここで使った Hooks の API は

-   useReducer : アプリケーションで管理する状態とアクションを格納
-   useMemo : 家電の MAX 登録件数かを判定する真偽値

useReducer の第 1 引数に reducer を、第 2 引数に initialState を渡して初期化を行います。  
state の取得は、initialState がオブジェクトになっているので、分割代入で取得をします。

useReducer の第 2 返却値の dispatch 関数は type プロパティを持つオブジェクトを受け取るようになっています。  
reducer 内で定義した action 関数を実行して得られるオブジェクトを渡しています。

## AddItem の編集

```tsx
// components/AddItem/index.tsx
import React, { useState, useEffect, useRef } from 'react';

type Props = {
    onAdd: (value: string) => void;
    onReset: () => void;
    disabled: boolean;
};

function AddItem({ onAdd, onReset, disabled }: Props) {
    const [textValue, handleChange] = useState('');
    const inputEl = useRef<HTMLInputElement>(null);
    const reset = () => handleChange('');

    useEffect(
        () => {
            if (disabled) return;
            const onEnter = (e: KeyboardEvent) => {
                if (e.keyCode === 13 && inputEl.current) {
                    const item = inputEl.current.value;
                    if (item) {
                        onAdd(item);
                        reset();
                    }
                }
            };

            window.addEventListener('keydown', onEnter);
            if (inputEl.current) {
                inputEl.current.focus();
            }
            return () => {
                reset();
                window.removeEventListener('keydown', onEnter);
            };
        },
        [disabled],
    );

    const _onHandleChange = (e: React.ChangeEvent<HTMLInputElement>) =>
        handleChange(e.target.value);
    const _onAddItem = () => {
        if (textValue) {
            onAdd(textValue);
            reset();
        }
    };

    return (
        <>
            <input
                ref={inputEl}
                type="text"
                onChange={_onHandleChange}
                disabled={disabled}
                value={textValue}
            />
            <button onClick={_onAddItem} disabled={disabled}>
                追加
            </button>
            {disabled && <button onClick={onReset}>リセット</button>}
        </>
    );
}

export default AddItem;
```

処理自体に大きな変更はありませんが、描画するエレメントやメモライズキーの設定などを行っています。

-   items を削除（reducer で管理）
-   テキストフィールドの初期化処理を onReset で別関数で切り出した
-   ボタンで家電を追加してもテキストフィールドが初期化されるようになった
-   家電が 5 件登録されたらイベントリスナーの登録を解除する

### 引数に設定するメモライズキーについて

AddItem では useEffect に親コンポーネントに渡ってくるメモライズキーを設定するようにしました。  
メモライズキーを渡すことで、指定されたキー値に変更があった時に useEffect の処理を発火させることができるようになります。

また、設定できるキー値は自コンポーネントの state だけではなく、
親コンポーネントから渡ってくる props や自コンポーネント内で管理される state ではないその他の値でも指定することができます。  
AddItem では、親コンポーネントから渡ってくる disabled をメモライズキーに指定をしています。

## ItemList の作成

続いて、登録された家電を表示する ItemList を作成します。  
components 配下に ItemList フォルダを作成してください。  
HowToTypeScript から大きな変更点がないのでコードの説明は省略します。

### ルートとなる index.tsx の作成

ItemList 配下に index.tsx を作成して下記コードを記述します。

```tsx
// components/ItemList/index.tsx
import React from 'react';
import styled from 'styled-components';
import Switch from './Switch';
import DisplayState from './DisplayState';
import { ItemProps } from '../reducer';

type Props = {
    onClick: (index: number) => () => void;
    items: ItemProps[];
};

function ItemList({ items, onClick }: Props) {
    return (
        <>
            {items.map(({ id, name, power }, index) => (
                <List key={id}>
                    <Item>{name}</Item>
                    <DisplayState power={power} />
                    <Switch onClick={onClick(index)} />
                </List>
            ))}
        </>
    );
}

const List = styled.div`
    width: 100%;
    display: flex;
    flex-wrap: nowrap;
    color: rgb(100, 100, 100);
    padding-bottom: 1rem;
    margin-bottom: 2rem;
    border-bottom: 1px solid rgb(100, 100, 100);
`;
const Item = styled.div`
    font-size: 3rem;
    width: 50%;
    text-overflow: ellipsis;
    white-space: nowrap;
`;
List.displayName = 'List';
Item.displayName = 'Item';

export default ItemList;
```

### DisplayState の作成

ItemList 配下に DisplayState.tsx を作成し下記コードを作成します。

```tsx
// components/ItemList/DisplayState.tsx
import React from 'react';

type Props = {
    power: boolean;
};

const styles = {
    width: '45px',
};

function DisplayState({ power }: Props) {
    return <div style={styles}>{power ? 'ON' : 'OFF'}</div>;
}

export default DisplayState;
```

### Switch の作成

ItemList 配下に Switch.tsx を作成し下記コードを作成します。

```tsx
// components/ItemList/Switch.tsx
import React from 'react';

type Props = {
    onClick: () => void;
};

function Switch({ onClick }: Props) {
    return <button onClick={onClick}>スイッチ</button>;
}

export default Switch;
```

これで、家電を表示できるようになりました。

## 現状の問題点

コードの作成はこれで終了です。  
Redux を使わずに家電管理アプリケーションを作成することができました。  
アプリケーションの動作自体には問題はありませんが、実はこのアプリケーションは不要なレンダリング処理が何回も走っています。

<img width="650" src="../imgs/HowToReactHooks/beforeRenderHighlight.gif">

上記アニメーションから、電源スイッチを押下した時と家電を追加した時に全ての要素が再レンダリングされていることがわかります。  
HowToRedux、HowToTypeScript はここで終了でしたが、HowToReactHooks では更に
パフォーマンスチューニングを<a href="./HowToReactHooks4">次のツアー</a>で行なっていきます。
