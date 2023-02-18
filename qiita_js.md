## はじめに

以前、JavaScript で配列同士の比較を行う方法について投稿しました。
[JavaScript で配列同士を比較して、重複・差分データを抽出する方法](https://qiita.com/uyupin/items/4ed7cfe205b411806413)

その際に素敵なコメントをいただき、別の方法により、
パフォーマンスの向上が見込めたので記します。

## 結論

Map・Set を使用して配列の比較を行う。

- Map
  キーと値のペアを保持するオブジェクト
- Set
  重複する値が存在しないコレクション

詳細は下記が非常に分かりやすかったため、参照してください。
[JavaScript Map オブジェクト](https://qiita.com/chihiro/items/9965cd7eca0380cf288c)
[JavaScript Set オブジェクト](https://qiita.com/chihiro/items/0e610a31b589e3cc435f)

## 事前準備

以下の二つの配列があったと仮定する。
「expense1」と「expense2」で差分データだけ抽出したい。

```javascript:main.js
const expense1 = [
  { name: '住居費', const: 50000 },
  { name: '水道光熱費', cost: 10000 },
  { name: '保険料', cost: 20000 },
  { name: '通信費', cost: 12000 },
  { name: '美容費', cost: 8000 },
];

const expense2 = [
  { name: '住居費', const: 50000 },
  { name: '教育費', cost: 15000 },
  { name: '水道光熱費', cost: 10000 },
  { name: '交際費', cost: 9000 },
  { name: '雑費', cost: 3000 },
];
```

## 実際に差分データだけ抽出する

```javascript:main.js
const expenseDiff = (expense1, expense2) => {

  const setExpense1 = new Set(expense1);
  let found = [];

  let map = new Map();

  expense1.forEach((cost) => map.set(cost.name, cost.name));
  //上記mapのkey,valueにexpense1のnameをセットする
  expense2.forEach((cost) => found.push(map.get(cost.name)));
  //expense2のnameとmapオブジェクトのkeyが同じ値を取得してfoundへpushする

  const setExpense2 = new Set(found);

  return new Set(
    [...setExpense1].filter((cost) => !setExpense2.has(cost.name))
    //setExpense1内にsetExpense2のnameが重複していないものを返却
  );
};

const result = Array.from(expenseDiff(expense1, expense2));
console.log(result);
```

結果 ↓

```javascript:main.js
[
  {name: '保険料', cost: 20000}
  {name: '通信費', cost: 12000}
  {name: '美容費', cost: 8000}
];
```

## 簡単なパフォーマンスの比較

以前投稿した方法(find や includes を使用する方法)と上記関数に"経過時間を測定する"処理を追加して比較
→ データ件数を 2,000 件に増加させております

#### 以前投稿した方法

```javascript:main.js
const arrayComparison = (expense1, expense2) => {
  const startTime = Date.now();
    let nowTime = 0;
    // 中略 //
  return (nowTime = (Date.now() - startTime) / 1000);
  //経過時間 0.042ms
};
```

#### Map・Set を使用した方法

```javascript:main.js
const expenseDiff = (expense1, expense2) => {
  const startTime = Date.now();
    let nowTime = 0;
    // 中略 //
  return (nowTime = (Date.now() - startTime) / 1000);
  //0.001ms
};
```

データ件数が少ない場合には、大きな違いにはなりませんが、大量のデータを扱う際には、Map・Set を使用した方が有効かもしれません。

## おわりに

`new Set`を使用した方がパフォーマンスが良いことがわかりました。
filter などは配列の全ての要素に対して処理を行うため、データ量の増加に対して処理も重くなります。
詳細は下記記事が非常に参考になるので、興味のある方はぜひ。
[JavaScript で重複排除を自分で実装してはいけない（Set を使う）](https://qiita.com/netebakari/items/7c1db0b0cea14a3d4419)
