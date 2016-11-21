---
title: エンティティの操作
template: tutorial-page.tmpl.html
tags: basics
---

このチュートリアルではエンティティの位置、方向、尺度を変更する方法を紹介します。

エンティティはPlayCanvasフレームワークを使用して構築されたほとんどのアプリケーションの基礎を形成します。エンティティは、プレイヤキャラクター、銃弾、敵、または単に空間内の点を表すことができます。

エンティティは特殊な形態のグラフノードです。`pc.GraphNode`から挙動の多くを継承します。以下で適用されるすべての操作はグラフノードにも適用することができます。

エンティティに実行する最も一般的な操作の1つは、変換行列の変更です。エンティティのローカル変換プロパティはエンティティの位置、方向及び尺度を決定し、同様にすべての子エンティティに影響を与えます。変換の操作方法を覚えるのは、インタラクティブなアプリケーションを作るために重要です。

### ローカル及びワールド座標

エンティティの移動及び操作を理解するには、ローカルとワールド座標系を理解する必要があります。ワールド座標系は全てのエンティティで共有され、固定の軸である`(0,0,0)`と、 `(0,1,0)` を上とする固定の方向があります。ローカルの座標系はエンティティ自体に比較します。つまり、ローカルの軸はエンティティの位置となり、方向はエンティティの方向に従います。

<img src="/images/tutorials/world.jpg" style="float:left;" alt="World co-ordinates"/>
<img src="/images/tutorials/local.jpg" style="float:right;" alt="Local co-ordinates"/>
<div style="clear:both" />
*ワールドとローカルの座標系*
<br />

### 階層

エンティティシステムについて理解しておくべき重要な部分は、エンティティグラフまたは階層です。エンティティはグラフノードタイプなので、グラフまたは親と子の階層に収集されています。各エンティティは、単一の親と複数の子を持つことができます。子エンティティは、親から変換情報を相続します。エンティティのワールド変換行列は、ローカルの変換を親エンティティのワールド変換と乗算します。例えば、子エンティティが`(1,0,0)`のローカル変換を持ち、その親が`(0,1,0)`のローカル変換を持つ場合、子のワールド位置は`(1,1,0)`になります。

## 位置

エンティティの位置を取得するのは簡単です

~~~js~~~
// エンティティの親の座標系に関連したエンティティの位置を取得
var lp = entity.getLocalPosition();

// ワールド空間でエンティティの位置を取得
var wp = entity.getPosition();
~~~

これらのメソッドはどちらも`pc.Vec3`(配列形式[x,y,z]のベクトル量) を返します。

エンティティの位置の設定は同じように簡単です。

~~~js~~~
// エンティティの親の座標系に関連させて、エンティティの位置を設定します
entity.setLocalPosition(x, y, z);

// ワールド空間内にエンティティの位置を設定します
entity.setPosition(x, y, z);
~~~

### エンティティを動かす

エンティティを移動するには、エンティティの位置を追加するか、translate と translateLocalのヘルパー関数を使用します。

~~~js~~~
// エンティティをワールド空間の正のX軸から1単位下に移す
entity.translate(1, 0, 0);

// エンティティをエンティティのローカルz軸から1単位下に移す
entity.translateLocal(0, 0, 1);
~~~

## 方向付け

エンティティの方向を設定するには、絶対的な回転を設定するか、インクリメンタルな回転を適用します。

絶対的な回転の設定は、 [Euler angles][1] または [quaternions][2]を使用して行うことができます。これら二つの数学的表現に対するWikipediaの説明は少し難しいですが、基本は簡単です。重要事項は次のとおりです：

** オイラー角 **

*オイラー角は、座標系のX,Y, Z軸を中心として（その順番通り）度単位の3つの回転です。
*座標系の軸を下に見ていくと、正のオイラー角は、その軸を中心とした反時計回りの回転となります。
*オイラー角は、その効果を思い浮かべることができるので、理解しやすいです。

** 4元数 **

*クォータニオンは4つの数字として格納され、3D空間内の任意の方向を表します。
*これらは、直接設定することは困難ですが、オイラー角、回転マトリックスまたは軸角表現から設定することができます。
*視覚化するのは難しいですが、堅牢であり、速やかに補間することができるので、(回転をアニメーション化する場合)便利です。

エンティティをスクリプトする場合、オイラー角を使用してエンティティの回転を設定する可能性が高いです。 例えば：

~~~js~~~
// 親エンティティの座標系のX軸を中心に反時計回りに30度回転してから
// Y軸を中心に45度回転して、最後に、Z軸を中心に60度回転します。
entity.setLocalEulerAngles(30, 45, 60);

// ワールド空間のX軸を中心に反時計回りに30度回転してから
// ワールド空間のY軸を中心に45度回転して、最後に、ワールド空間のZ軸を中心に60度回転します。
entity.setEulerAngles(30, 45, 60);
~~~
しかし、エンティティの回転を四元形式で設定したい場合、次の何れかの関数を利用できます：

~~~js~~~
// アイデンティティ回転を作成
var q = new pc.Quat();
// エンティティをその親と同じ回転を持つよう設定する。
// entity.setLocalEulerAngles(0, 0, 0)
entity.setLocalRotation(q);

// Set the entity to have no rotation with respect to the world space coordinate
// system  - equivalent to entity.setEulerAngles(0, 0, 0)
entity.setRotation(q);
~~~

インクリメンタルにエンティティを回転するには、rotateを使用してエンティティをワールド空間軸に関連させて回転させるか、rotateLocalを使用してエンティティの現在の軸に関連させて回転させます。

例えば、ワールド上軸の周りに180度エンティティを回転させるには：

~~~js~~~
entity.rotate(0, 180, 0);
~~~

または、ローカルX軸の周りにエンティティを90度回転させるには：

~~~js~~~
entity.rotateLocal(90, 0, 0);
~~~

## スケール

エンティティを拡大縮小するには次の関数を呼び出します：

~~~js~~~
// ローカルY軸でエンティティを2の倍数でスケール
entity.setLocalScale(1, 2, 1);
~~~

もう少し興味深い例を紹介します：

~~~js~~~
// 時間上でsine関数を使用してエンティティをスケール
this.timer += deltaTime;
var s = Math.sin(this.timer) + 1;
entity.setLocalScale(s, s, s);
~~~

現在、ワールド空間でエンティティのスケールを設定することはできません。

[1]: http://en.wikipedia.org/wiki/Euler_angles
[2]: http://en.wikipedia.org/wiki/Quaternion
