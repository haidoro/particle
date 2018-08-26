# パーティクル作成
(パーティクル作成)[https://github.com/haidoro/particle]
## 円形の描画
### canvasサイズの指定(GitHub drawブランチ)
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

### ブラウザの幅いっぱいにcanvasを広げ、リサイズ対応
canvas幅と高さをいっぱいに広げます。これはCSSで「width: 100vw」「height: 100vh」にすることで対応します。

また、リサイズに対応させるためには次のように「resize()」関数を作成しておきます。

```
function resize() {
      stageW = innerWidth * devicePixelRatio;
      stageH = innerHeight * devicePixelRatio;
      canvas.width = stageW;
      canvas.height = stageH;
}
```
そのまま「resize()」関数を実行しても、リサイズするごとに再読み込みを実行しなければなりません。画面をリサイズすると自動でアップデートする仕組みは次のように「tick()」関数を作成して対応しています。後は、addEventListenerでresizeイベントを感知するとresize関数を実行します。

```
function tick() {
  requestAnimationFrame(tick);
  // 画面を更新する
  draw();
}
```

```
window.addEventListener('resize', resize);
```

## パーティクル発生の準備（particle-objectブランチ）

パーティクル発生に必要な条件を「particle」オブジェクトとして作成。ここでは発生させる場所の座標を決めています。  
「particle」オブジェクトはさらに「particles」配列に入れて関数化しておきます。こうすることで定期的に関数を実行すると、パーティクル発生条件である「particle」オブジェクトを複数作成することができるようになります。

```
function emit() {
      // オブジェクトの作成
      const particle = {
        x: stageW / 2, // パーティクルの発生場所(X)
        y: stageH * 4 / 5, // パーティクルの発生場所(Y)
      };
      // 配列に保存
      particles.push(particle);
    }
```

当然draw関数に記述した円形作成メソッドarcのパラメータも変更しておきます。

```
function draw() { 
        context.beginPath();
        // 色を設定
        context.fillStyle = 'red';
        // 円を描く
        context.arc(particles[0].x, particles[0].y, 50, 0, (Math.PI/180)*360, false);
        // 形状に沿って塗る
        context.fill();
        context.closePath();
    }
```
これで画面中央の下部に円形が一つ出現します。

## 複数の円形を発生
`setInterval(emit, 1500)`を使うことで1.5秒ごとに`particle`オブジェクトを作り続けて`particles`に格納していきます。

パーティクルの更新は次のようにします。  
1. forループ文でparticles配列に格納されたparticleオブジェクトを取り出します。
2. 重力は速度（vy）に1を減算し、摩擦は速度（vy）の0.92バイトし、それぞれ代入します。
3. 速度は`particle.y += particle.vy`で実現できます。
4. 寿命はMAX_LIFEの値を1ずつ減算して0になったら、splice()で`particles`配列から削除します。

```
    function update() {
      // パーティクルの計算を行う
      for (let i = 0; i < particles.length; i++) {
        // オブジェクトの作成
        const particle = particles[i];
        // 重力
        particle.vy -= 1;
        // 摩擦
        particle.vy *= 0.92;
        // 速度を位置に適用
        particle.y += particle.vy;
        // 寿命を減らす
        particle.life -= 1;
        // 寿命の判定
        if (particle.life <= 0) {
          // 配列からも削除
          particles.splice(i, 1);
          i -= 1;
        }
      }
    }
```

draw()関数も少し手を加えます。  
forEach文で`particles`配列から`particle`を一つずつ抜き出して円形の描画を行います。
この内容をアロー関数を使用して記述しています。

```
function draw() { 
      context.clearRect(0, 0, stageW, stageH);
      
      particles.forEach(particle => {
        context.beginPath();
        // 色を設定
        context.fillStyle = 'red';
        // 円を描く
        context.arc(particle.x, particle.y, 50, 0, (Math.PI/180)*360, false);
        // 形状に沿って塗る
        context.fill();
        context.closePath();
      });
    }
```