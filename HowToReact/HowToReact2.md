# React Component について

先ほどのチュートリアルではコンポーネントとは UI 部品とお伝えしました。  
部品として、どの程度の粒度で分けるかは、コンポーネントの規模によりますが、基本的に再利用可能な粒度にとどめるのが良いです（気になった方は、Atomic Design という理論を参考にしてみてください）  
<br />

先ほどのチュートリアルによって画面に文字列を表示できるようになりました。
しかし、単一のコンポーネントを表示させるだけではウェブページと呼べません。  
ここで、コンポーネントを合成することが必要になってきます。

コンポーネントを合成し、合成されたコンポーネントを他のコンポーネントに合成し……を繰り返して、1 つのページが完成します。  
コンポーネントがどの程度の階層を持っているかは、作りたい部品やプロジェクトの方針によって異なりますが、大まかにはページを構成するコンポーネントの上に UI 部品を構成するコンポーネントを乗せていくイメージになります。  
<br />

また、コンポーネントには I/O があります。  
「コンポーネントは与えられたデータに従って、振る舞いを変える」という挙動をコーディングすることができます。  
<br />
このチュートリアルでは

-   コンポーネントの合成
-   データフローを扱う（コンポーネントの I/O を利用する）

の 2 つの項目を扱います。

# 成果物

<img src='../img/HowToReact/reactTour2.jpeg' />

## コンポーネントを合成する

次に先ほどの Hello React World! を再利用可能なコンポーネントとして切り出します。  
src フォルダに HRC.js というファイルを新規作成してください。

```jsx
// HRC.js
import React, { Component } from 'react';

export default class HRC extends Component {
    render() {
        return <div>Hello React World!</div>;
    }
}
```

ルートコンポーネントである App.js を下記のように変更します。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import HRC from './HRC';

class App extends Component {
    render() {
        return (
            <div>
                <HRC />
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

上記コードでは、App.js の render メソッドで return する node に &lt;HRC /&gt; というシンタックスで HRC コンポーネントが内包されました。これがコンポーネントの合成です。  
このように React ではネスト構造にすることでコンポーネントを合成することができます。  
この場合、App コンポーネントの中に HRC コンポーネントがネストされています。

コンポーネントの合成では

-   コンポーネントを読み込む
-   読み込んだコンポーネントを <コンポーネント名 /> のように記述する

ということがポイントとなります。  
<br />

### 構文解説

#### export default

```jsx
export default class .....略
```

という構文が HRC.js には出現しました。  
これは class を出力するために必要な構文です。

```
import HRC from './HRC';
```

にて他ファイルからも、コンポーネントを読み込めるようになります。

#### export と export default の違い

どちらの構文も記述することによって、import できます。  
また、1 ファイル内に両方の記述があっても共存できます。

##### 出力時

```
export default ・・・・
```

はデフォルトで読み込む対象を指定することができます。ただし、1 ファイルに 1 回しか使うことはできません。  
export default は import する際に特に指定しなければ export default で指定されたものを読み込むようになります。

一方

```
export ・・・・
```

は複数使うことができます。

##### 読み込み時

export default を使ったコンポーネントの読み込み

```
import HRC from './HRC'
// <HRC />で使用可能
```

export を使ったコンポーネントの読み込み

```
import { HRC } from './HRC';
// <HRC />で使用可能
```

or

```
import HRC from './HRC';
// <HRC.HRC />で使用可能 *
```

\* ただし、export default が読み込むファイルで宣言されていると、export default されたモジュールのプロパティ HRC を参照してしまうので `<HRC.HRC />` という構文は使えません。  
対処法として、下記のようにモジュール全体を HRC という変数名で読み込むことが出来ます。

```
import * as HRC from './HRC';
// <HRC.HRC />で使用可能
```

##### 例

定数のみを定義したファイル constants.js が下記のように記述されているとします。

```jsx
// constants.js
export const FIRST = '1';
export const SECOND = '2';
export default const THIRD = '3';
export const FORTH = '4';
```

constants.js の定数を読み込む場合、default 句で指定された THIRD はそのまま記述し、default 句を指定していない FIRST, SECOND は{}で囲んで読み込みます。import で指定していない FORTH は読み出す側では定義していないので、undefined になります。

```jsx
import THIRD, { FIRST, SECOND } from './constants';

console.log(FIRST); // 1
console.log(SECOND); // 2
console.log(THIRD); // 3
console.log(FORTH); // undefined
```

## データフローを扱う

Angular1.X などに代表される MVC アーキテクチャではデータの流れといった秩序は乏しく、データフローは複雑でした。  
それに対して React のデータフローは、親から子への単方向のみという非常に単純な仕組みによって、秩序を保っています。  
詳細については props の説明の後で行います。

### props について

React の非常に重要なポイントで、props という仕組みがあります。  
props とは親から渡されるプロパティ群のことで、props を使うことによって、各コンポーネントに対する Input を与えることができます。

### コンポーネントツリーについて

基本的に React を使った開発でコンポーネントはツリー構造になります。

```
     App
    /   ＼
   HRC   HRC2
  /          |  ＼  ＼
 granC  AA  BB CC
  |
 gGranC
```

この場合、App.js の render メソッドは下記のようになります。

```jsx
// App.js
...
render(){
    return (
        <div>
            <HRC />
            <HRC2 />
        </div>
    );
}
```

また、HRC2 コンポーネントの render メソッドは下記のようになります。

```jsx
//HRC2.js
...
render(){
    return (
        <div>
            <AA />
            <BB />
            <CC />
        </div>
    );
}
```

以上のことから、コンポーネントをネストすることによってツリー構造が生まれることが分かっていただけたかと思います。

### ツリー構造とデータフロー

React では、このツリー構造の親から子へのみの単方向のデータフローのみが生じます。そのため、こちらの例では、

```
     App
    /   ＼                     |
   HRC   HRC2           | data-flow
  /           |＼ ＼         V
 granC  AA  BB CC
  |
 gGranC
```

-   App から HRC にデータ渡す ○
-   HRC から granC にデータ渡す ○
-   APP から AA にデータ渡す ○（App から HRC2 に渡して、HRC2 から AA に渡す）

上記のパターンは問題なく実装することができます。  
反対に

-   granC から APP にデータを渡す ×
-   AA から BB にデータを渡す ×
-   gGracC から App にデータを渡す ×

というように直接の親子関係で親から子のパターン以外では、データの受け渡しはできません。  
このように React では親から子という単方向データフローという秩序があります。  
応用として、Redux を使うことで秩序を保ちつつより柔軟なデータフローを実現することができますが、このページでは React 単体について解説していきます。

<br>
Redux については<a href="../HowToRedux/HowToRedux1">こちら</a>をご確認ください。

### props を受け取る

先ほど「コンポーネントを合成する」で作成した HRC コンポーネントに変更を加えます。
`export default class` が `export default HRC` になったことに注意してください。また、新しく PropTypes というモジュールを import しています。

```
// HRC.js
import React, { Component } from 'react';
import PropTypes from 'prop-types';

class HRC extends Component {
    render() {
        const { title, children } = this.props;
        return (
            <div>
                <div>{title}</div>
                <div>{children}</div>
            </div>
        );
    }
}
HRC.defaultProps = {
    title: '',
    children: null,
};
HRC.propTypes = {
    title: PropTypes.string,
    children: PropTypes.node,
};

export default HRC;
```

このコンポーネントは親コンポーネント（App コンポーネント）から title, children の 2 つの props を受け渡されることを期待しています。

親から受け取った props は this.props にて参照することができます。

```js
const { title, children } = this.props;
```

この場合は、this.props に含まれる title, children プロパティをそれぞれ、title, children として定数宣言しています。
また jsx 内では {} というシンタックスで囲われたものは JavaScript として評価されます。そのため、定数の値を出力させるために

```jsx
    <div>{title}</div>
    <div>{children}</div>
```

このように {} で括って出力させます。

#### defaultProps

React Component では defaultProps を指定することができます。これは、親から props が渡ってこなかった場合に使用する値です。

defaultProps は下記のように定義します。

```
HRC.defaultProps = {
    title: '',
    children: null,
};
```

こうすることで、this.props に title が含まれない場合は''（空文字）、children が含まれない場合は null がセットされます。

#### propTypes

React Component では propTypes を設定することができます。PropTypes は期待する props のデータ型を記述することができます。

```
HRC.propTypes = {
    title: PropTypes.string,
    children: PropTypes.node,
};
```

この場合では、title は文字列、children は node（DOM や文字列といった画面に出力できるもの）が指定されています。  
もしも、予想外のデータ型の props が入ってきた場合 warning として出力されます（アプリケーションは動き続けます）。  
defaultProps とともに propTypes を記載することで、受け取る props の関係が分かりやすくなります。  
production ビルドの場合は warning として出力されないので、確認は development ビルドで行って下さい。

### props を渡す Attribute

次に HRC コンポーネントの親コンポーネントである App.js を変更します。

```jsx
// App.js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import HRC from './HRC';

class App extends Component {
    render() {
        return (
            <div>
                <HRC title="React Title">this is this.props.children</HRC>
            </div>
        );
    }
}

ReactDOM.render(<App />, document.getElementById('content'));
```

親コンポーネントから子コンポーネントへの値の受け渡し方は非常に単純で、HTML 属性のように

```
    prop名=値
```

の形で受け渡します。そのためこのサンプルでは

```jsx
    <HRC title="React Title" ...略
```

とすることで HRC コンポーネントに title を受け渡しています。  
これを受け取った HRC コンポーネントでは、this.props.title で参照される値が React Title になります。

### props を渡す children

this.props.children という予約語を渡すためには HTML 属性のようには渡しません。

```jsx
<HRC children="this is ..." /> // NG
```

下記のように HRC の子要素として受け渡すことによって成立します。

```jsx
    <HRC ...略>
        this is this.props.children
    </HRC>
```

この場合は、HRC コンポーネントでは、this.props.children = this is this.props.children という文字列になります。
