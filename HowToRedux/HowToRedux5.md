# redux-logger を使用する

このページでは外部パッケージを使って、開発を効率的に進めていく方法を解説します。  
action が発行されると state が変化するというのが Redux における基本的な流れになります。  
開発時には何かと追って行きたいことが多いですがこれを追うためには事細かに console.log を仕込まなければなりません。

<a href='https://github.com/evgenyrodionov/redux-logger' target='_blank'>redux-logger</a> を使用することで、見やすく、なおかつ簡単に追うことが可能になります。

## redux-logger を設定する

redux-logger は Middleware に設定します。
middlewares/index.js を編集し、applyMiddleware に logger を追加します。

```js
// middlewares/index.js
import { applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
import changePowerState from './itemList';

const logger = [];
if (process.env.NODE_ENV === 'development') {
    logger.push(createLogger());
}

const middlewares = applyMiddleware(changePowerState, ...logger);

export default middlewares;
```

以上の設定が終わりましたら後は、ブラウザで F12 キーを押下してコンソールを眺めるだけで確認できます。( mac で chrome をお使いの方は「cmd + alt + I」)  
<br>
<img src='../img/HowToRedux/reduxLogger.gif' />

#### process.env.NODE_ENV について

これは環境変数になります。  
開発サーバ（webpack）が、起動時に

`NODE_ENV = 'development'`

という環境変数を自動付与してくれており、これを利用しております。  
production としてリリースする際には、この NODE_ENV が production という文字列に変わるので、
リリースされた React アプリケーションでは、redux-logger のログが出ないということができます。

# formik を使用する

先ほどのページで行った入力値の管理は非常に手間に感じたかもしれません。入力値を Store で管理させるためには以下の手順を行う必要があります。

-   入力イベントに対して dispatch する関数を用意
-   対応する action を用意
-   対応する reducer を用意
-   一意のイベントキーを設定

1 つテキストフィールドがあるだけでこれだけの準備が必要になります。そのため、入力フォームが多数あるページではとても実装しきれません。
そこで、<a href='https://jaredpalmer.com/formik/docs/overview' target='_blank'>formik</a> を使うことでこれらの問題を解決することができます。

## formik を設定する

formik はコンポーネントに対して直接設定していきます。  
compoents/AddItem.js を編集します。

```jsx
// components/AddItem.js
import React, { Component } from 'react';
import { withFormik } from 'formik';
import * as Yup from 'yup';
import PropTypes from 'prop-types';

class AddItem extends Component {
    static defaultProps = {
        textChangeAction() {},
        addItemAction() {},
        textValue: '',
    };

    render() {
        const { handleSubmit, handleChange, handleBlur, errors, values } = this.props;
        return (
            <form onSubmit={handleSubmit}>
                <input
                    type="text"
                    name="item"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.item}
                />
                {errors.item && <span>{errors.item}</span>}
                <button type="submit">追加</button>
            </form>
        );
    }
}
TextField.propTypes = {
    textChangeAction: PropTypes.func,
    addItemAction: PropTypes.func,
    textValue: PropTypes.string,
    handleSubmit: PropTypes.func,
    handleChange: PropTypes.func,
    handleBlur: PropTypes.func,
    errors: PropTypes.shape({
        item: PropTypes.string,
    }),
    values: PropTypes.shape({
        item: PropTypes.string,
    }),
};
export default withFormik({
    mapPropsToValues: () => ({
        item: '',
    }),
    validationSchema: () =>
        Yup.object().shape({
            item: Yup.string().required('入力必須です'),
        }),
    handleSubmit: (values, { props, resetForm }) => {
        const { items, addItemAction } = props;
        const addedItems = [
            ...items,
            {
                id: `item-index-of-${items.length}`,
                name: values.item,
                power: false,
            },
        ];
        addItemAction(addedItems);
        resetForm();
    },
    displayName: 'itemForm',
})(AddItem);
```

formik との結合を HOC か renderProp で行うことができますが、ここでは HOC パターンで紹介します。  
formik の大まかな設定は下記のように export default 時に設定をします。

```js
export default withFormik({
    mapPropsToValues: () => ({
        item: '',
    }),
    validationSchema: () =>
        Yup.object().shape({
            item: Yup.string().required('入力必須です'),
        }),
    handleSubmit: (values, { props, resetForm }) => {
        const { items, addItemAction } = props;
        const addedItems = [
            ...items,
            {
                id: `item-index-of-${items.length}`,
                name: values.item,
                power: false,
            },
        ];
        addItemAction(addedItems);
        resetForm();
    },
    displayName: 'itemForm',
})(AddItem);
```

withFormik 関数は様々なプロパティを設定することができ、ここで設定しているのは以下の 4 つです。

|   プロパティ名   | コールバックの引数  |                      説明                       |
| :--------------: | :-----------------: | :---------------------------------------------: |
| mapPropsToValues |       (props)       |             フォームの初期化を行う              |
| validationSchema |       (props)       | Yup（後述します）による validation の設定を行う |
|   handleSubmit   | (values, formikBag) |              submit 時の処理を行う              |
|   displayName    |        なし         |               入力フォームの名前                |

|  引数名   |                     説明                      |
| :-------: | :-------------------------------------------: |
|   props   |        コンポーネントが受け取る props         |
|  values   | formik が管理する入力値（上記コードだと item) |
| formikBag |          formik オブジェクトと props          |

### mapPropsToValues

このメソッドでフォームの初期化と、コンポーネント内でフォーム入力値を管理するオブジェクトの作成を行います。  
コールバックで props を受け取ることができ、props を元にフォームの初期化を行うこともできます。  
作成されたオブジェクトは values という props で参照することができます。

### validationSchema

Yup を使用して validation をかける際に設定するメソッドになります。  
mapPropsToValues と同様に props を受け取ることができ、props を元に validaton をかけることができます。  
もし、Yup で設定した項目を満たさない場合、オブジェクトが返却され、errors という props で参照することができます。

### handleSubmit

submit となるボタンやアクションをトリガーにして実行されるメソッドです。  
主に入力値を引数に Action を発行する、コンポーネントの後始末（フォームリセットやダイアログのクローズ）などを行います。

```js
const { items, addItemAction } = props;
const addedItems = [
    ...items,
    {
        id: `item-index-of-${items.length}`,
        name: values.item,
        power: false,
    },
];
addItemAction(addedItems);
resetForm();
```

前回のツアーでは middleware 内で行なっていた処理を component 内で行うように変更しました。
validation エラーがあった場合、handleSubmit の処理は実行されませんので、if 文を挟むことなく  
スマートに記述できるようになりました。  
最後に resetForm という formikBag の関数を利用して、フォームリセットをここでは行なっています。

### formik の props の取得

formik の props(formikBag)は、コンポーネント内のどこからでも受け取ることができ、  
コード中では

-   handleSubmit : フォーム送信メソッド
-   handleChange : フォーム入力値を変更するメソッド
-   handleBlur : フォーカスされた挙動を検知するメソッド
-   errors : formik に登録されたフォームオブジェクトのエラー
-   values : formik に登録されたフォームオブジェクト

この 5 つを利用しています。

### input エレメントとの連携

input と formik を接続するには、
value 属性にフォームオブジェクトのキー値と
name 属性に紐付けたいフォームオブジェクトのキー名を name 属性に与えます。

```jsx
<input type="text" name="item" onChange={handleChange} onBlur={handleBlur} value={values.item} />
```

handleChange メソッドを渡すことで、入力値の変更を監視してくれます。
handleBlur はエラーをオブジェクトへの反映を行わせるために渡しています。

### Yup について

formik は validation を設定することをできますが、Yup というモジュールを使うことで
簡単に validaton をかけることができます。
公式でも validation には Yup を推奨しています。

```js
Yup.object().shape({
    item: Yup.string().required('入力必須です'),
}),
```

ここでは item キーに

1. 文字列型で
1. 入力必須項目である

という validation をかけています。  
もし、何も入力せずに追加ボタンを押すと

```js
errors = {
    item: '入力必須です',
};
```

という値がセットされます。
以上で、この章での作業は終わりになります。

## 追記（2018/11/21)

このマークダウンを書いた時は formik を HOC で紹介していますが、これは renderProps の方がいいです。  
修正が間に合っていません、すみません。。。

React Hooks の登場で、recompose という HOC を簡単に作ることができるモジュールの新規開発が打ち切りになりました。  
HOC はわかりづらくしてしまうのが理由です。  
なので、formik は基本的に renderProps で記述するのが望ましいです。

筆者（camcam_lemon）は、HOC も renderProps もどちらも使います。  
パフォーマンスチューニングやイベントをトリガーにしたフォーム変更や API 通信を行う場合は HOC の方が良いと考えています。  
詳しくは<a href='https://www.figma.com/file/NjhPThPgXm1CYuUEHqjWct6G/Formik%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8B%E5%85%A5%E5%8A%9B%E5%80%A4%E7%AE%A1%E7%90%86' target='_blank'>こちら</a>をご覧ください。
