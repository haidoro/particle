# パーティクル作成
## 円形の描画
### canvasサイズの指定
canvasのサイズを指定は少し面倒です。  
canvasのサイズを変更する場合、CSSでwidthとheightを指定すると、canvasのデフォルトサイズからの相対サイズになってしまうため、canvas内に描画した画像が引き伸ばされることになります。  
そのため、CSSで対応するにはcanvasタグにラッパー要素を作り、その要素に幅と高さを決めます。

または、JavaScriptで対応する方法として次のようにします。この場合ラッパー要素は必要ありません。

サイズを980x480にした例
```
const canvas = document.querySelector('canvas');
canvas.width = 980;
canvas.height = 480;
```

### 円形の描画方法
canvasに描画する場合は必ず「canvas.getContext('2d')」が必要です。
これは描画の際決まりごととして記述するようにします。

円形の描画は「arc()」メソッドを使います。「arc()」メソッドは円や円弧を描きます。  
パラメータは次の通りです。

```
arc(x, y, radius, startAngle, endAngle, anticlockwise)
```

x
円の中心のx座標
y
円の中心のy座標
radius
円の半径
startAngle
円弧を描き始める角度。x軸の向き（右方向）からみて右回りの角度をラジアンで指定。  
円の場合は0です。
endAngle
円弧を描き終える角度。x軸の向き（右方向）からみて右回りの角度ラジアンで指定。  
円の場合は「(Math.PI/180)*360」です。
anticlockwise
円弧を描く向きを真偽値で指定。trueを指定すると反時計回り（左回り）に、falseを指定すると時計回り（右回り）で円弧を描きます。

円形の描画は次のようになります。
```
const context = canvas.getContext('2d');
context.beginPath();
context.fillStyle = 'red';
context.arc(100, 100, 50, 0, (Math.PI/180)*360, false);
context.fill();
context.closePath();
```
`fillStyle`で色を決めて、`fill()`で着色します。
`beginPath()`で描画を始め、`closePath()`で終了します。

