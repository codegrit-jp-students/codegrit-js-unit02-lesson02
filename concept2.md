## コールバック関数

コールバック関数とは、ある関数(関数Aと呼ぶ)の引数として与えられる関数で、関数Aが一定の処理をしてから呼び出されることから、コールバック(呼び戻し)関数と呼ばれます。

例えば、以下がその例です。

```javascript
function logSomething(something) {
  console.log(something);
}

function callName(name, action) {
  let something = "Hello " + name;
  return action(name);
}

callName("World", logSomething);
```

実行結果: 
```
Hello World
```

この例ではcallNameという関数の2つ目の引数にlogSomethingという関数を与えています。このlogSomethingをコールバック関数と呼びます。またcallName関数のように別の関数を返す関数のことを高次関数(Higher-order Function)と呼びます。

## コールバック地獄

ES6以前のJavascriptでは非同期処理を書くにはコールバックを使うしかありませんでした。このコールバックは簡単な処理であればそれほど問題ないのですが、複数の非同期処理を連結して行う必要があるような場合、コールバック地獄と呼ばれる問題を引き起こしてしまいます。

例えば、以下ではfuncAという関数の結果を使ってfuncBを呼び出し、更にその結果を使ってfuncCを呼び出すfという関数を定義しています。

```javascript
function f() {
  return funcA(arg1, (err, res1) => {
    return funcB(res1, (err, res2) => {
      return funcC(res2, (err, res3) => {
        console.log(res3);
      }
    })
  })

}
```

上のように、関数の実行結果を次々と新しいコールバック関数に利用しているため、どんどんとネストが深くなってしまっています。上記の例は非常に単純化しており、エラー処理も入っていませんが、より複雑なプログラムだとどういう処理をしているのかが非常に把握しづらくなります。