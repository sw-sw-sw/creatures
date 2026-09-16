# Particle Creature

p5.jsのコードゴルフ作品を、WebGL2(頂点シェーダー)に移植した生成アート。1万個の点を三角関数と冪乗で変形させ、16体の「生き物」のような群れを描画する。画面右側のスライダーで各パラメーターをリアルタイムに調整できる。

https://sw-sw-sw.github.io/creatures/

## 仕組み

各点の座標は、点ごとの固定ID `i`(0〜9999)と時刻 `t` だけから毎フレーム直接計算される。前フレームの出力を次の入力に戻すような反復(フィードバック)はなく、いわゆるカオス系のアトラクタとは異なる仕組み。

- `i % 16` で1万個の点を16グループに分け、グループごとに個体(創発する「生き物」)を形成
- `cos(i*5)*sin(i)` のように大きな整数をそのまま三角関数に渡すことで、決定論的ながら疑似ランダムな枝分かれ・ばらつきを生成
- 半径 `d` を `d ** sin(...)` という冪乗で変調し、有機的な「呼吸」のようなパルスを作る
- 異なる周波数の `sin`/`cos` を重ね合わせる点はハーモノグラフ/リサージュ曲線と同じ原理

## 元コード(p5.js)

```js
a=(m,d=mag(k=9*cos(i*5)*sin(i),e=cos(i*3)*cos(i*2)*9)**3/1999+1.5-sin(t/2+m)**3/3)=>
  point(99*sin(c=d/16-t/48+m)+k*(p=d**sin(d*d-t+m))+200,
        99*sin(c*4)+e*p+200)
t=0,draw=$=>{
  t||createCanvas(w=400,w);
  background(9).stroke(w,96);
  for(t+=PI/20,i=1e4;i--;) a(i%16*13)
}
```

## WebGL(GLSL)コアの移植部分

`i` は `gl_VertexID` に置き換え、10000頂点を `gl.POINTS` として1回の `drawArrays` で描画する。定数はスライダー用の `uniform` に置き換えている。

```glsl
float i = float(gl_VertexID);
float m = mod(i, uGroups) * uSpacing;      // 元コード: i % 16 * 13

float k = uSpike * cos(i * 5.0) * sin(i);
float e = cos(i * 3.0) * cos(i * 2.0) * uSpike;
float mg = sqrt(k * k + e * e);            // 元コード: mag(k, e)

float d = pow(mg, 3.0) / 1999.0 + 1.5 - pow(sin(uT / 2.0 + m), 3.0) / 3.0;
float c = d / 16.0 - uT / uTwist + m;
float p = pow(d, sin(d * d - uT + m));     // 元コード: d ** sin(...)

float x = uAmp * sin(c) + k * p + 200.0;
float y = uAmp * sin(c * uYFreq) + e * p + 200.0;
```

## 元の引用元

https://x.com/yuruyurau/status/2090832898488459699?s=20
