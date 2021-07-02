
<h1 align="center">Backgammon OpenAI Gym</h1> <br>
<p align="center">
   <img alt="Backgammon" title="Backgammon" src="./logo.png" width="350">
</p>

# Table of Contents
- [gym-backgammon](#gym_backgammon)
- [インストール](#installation)
- [環境](#env)
    - [観測](#observation)
    - [Actions](#actions)
    - [報酬](#reward)
    - [初期配置](#starting_state)
    - [エピソードの終了](#episode_termination)
    - [リセット](#reset)
    - [レンダリング](#rendering)
    - [例](#example)
        - [Play random Agents](#play)
        - [有効なアクション](#valid_actions)
- [Backgammon Versions](#versions)
- [Useful links and related works](#useful_links)
- [License](#license)

---
## <a name="gym_backgammon"></a>gym-backgammon
バックギャモンゲームは、チェッカーの動きとサイコロの目を使った2人用のゲームです。各プレイヤーの目標は、自分のチェッカーをすべてボードから移動させることです。 

このリポジトリには、OpenAI Gymでのバックギャモンゲームの実装が含まれています。  
現在のボードの状態、サイコロの目、現在のプレイヤーが与えられると、現在のプレイヤーが実行できるすべての有効なアクション/ムーブを（反復的に）計算します。 有効なアクションは、その状態とプレイヤーに対して（可能であれば）最高のサイコロの目を使用するように生成されます。

---
## <a name="installation"></a>インストール
```
git clone https://github.com/dellalibera/gym-backgammon.git
cd gym-backgammon/
pip3 install -e .
```

---
## <a name="env"></a>環境
状態を表現するためのエンコーディングは、Gerald Tesauro[1]が使っていたものを参考にしています。

### <a name="observation"></a>観測
Type: Box(198)

| Num       | Observation                                                         | Min  | Max  |
| --------- | -----------------------------------------------------------------   | ---- | ---- |
| 0         | WHITE - 1st point, 1st component                                    | 0.0  | 1.0  |
| 1         | WHITE - 1st point, 2nd component                                    | 0.0  | 1.0  |
| 2         | WHITE - 1st point, 3rd component                                    | 0.0  | 1.0  |
| 3         | WHITE - 1st point, 4th component                                    | 0.0  | 6.0  |
| 4         | WHITE - 2nd point, 1st component                                    | 0.0  | 1.0  |
| 5         | WHITE - 2nd point, 2nd component                                    | 0.0  | 1.0  |
| 6         | WHITE - 2nd point, 3rd component                                    | 0.0  | 1.0  |
| 7         | WHITE - 2nd point, 4th component                                    | 0.0  | 6.0  |
| ...       |                                                                     |      |      |
| 92        | WHITE - 24th point, 1st component                                   | 0.0  | 1.0  |
| 93        | WHITE - 24th point, 2nd component                                   | 0.0  | 1.0  |
| 94        | WHITE - 24th point, 3rd component                                   | 0.0  | 1.0  |
| 95        | WHITE - 24th point, 4th component                                   | 0.0  | 6.0  |
| 96        | WHITE - BAR checkers                                                | 0.0  | 7.5  |
| 97        | WHITE - OFF bar checkers                                            | 0.0  | 1.0  |
| 98        | BLACK - 1st point, 1st component                                    | 0.0  | 1.0  |
| 99        | BLACK - 1st point, 2nd component                                    | 0.0  | 1.0  |
| 100       | BLACK - 1st point, 3rd component                                    | 0.0  | 1.0  |
| 101       | BLACK - 1st point, 4th component                                    | 0.0  | 6.0  |
| ...       |                                                                     |      |      |
| 190       | BLACK - 24th point, 1st component                                   | 0.0  | 1.0  |
| 191       | BLACK - 24th point, 2nd component                                   | 0.0  | 1.0  |
| 192       | BLACK - 24th point, 3rd component                                   | 0.0  | 1.0  |
| 193       | BLACK - 24th point, 4th component                                   | 0.0  | 6.0  |
| 194       | BLACK - BAR checkers                                                | 0.0  | 7.5  |
| 195       | BLACK - OFF bar checkers                                            | 0.0  | 1.0  |
| 196 - 197 | Current player                                                      | 0.0  | 1.0  |

1つのポイントのエンコード（そのポイントにいるチェッカーの数を示す）:

| Checkers | Encoding                                |           
| -------- | --------------------------------------- |
| 0        | [0.0, 0.0, 0.0, 0.0]                    |
| 1        | [1.0, 0.0, 0.0, 0.0]                    |
| 2        | [1.0, 1.0, 0.0, 0.0]                    |
| >= 3     | [1.0, 1.0, 1.0, (checkers - 3.0) / 2.0] |

BARチェッカーのエンコード:

| Checkers | Encoding             |           
| -------- | -------------------- |
| 0 - 14   | [bar_checkers / 2.0] |

OFFバーチェッカーのエンコード:

| Checkers | Encoding              |           
| -------- | --------------------- |
| 0 - 14   | [off_checkers / 15.0] |

現在のプレーヤーのエンコード:

| Player  | Encoding   |           
| ------- | ---------- |
| WHITE   | [1.0, 0.0] |
| BLACK   | [0.0, 1.0] |

### <a name="actions"></a>Actions
エージェントが実行できる有効なアクションは、現在の状態とサイコロの出目に依存します。そのため、行動空間の形は決まっていません。 

### <a name="reward"></a>報酬
`WHITE`が勝ったら`1`、`BLACK`が勝ったら`0`

### <a name="starting_state"></a>初期配置
すべてのエピソード/ゲームは同じ場所から始まります。（[starting position](#starting_position)）: 
```
| 12 | 13 | 14 | 15 | 16 | 17 | BAR | 18 | 19 | 20 | 21 | 22 | 23 | OFF |
|--------Outer Board----------|     |-------P=O Home Board--------|     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |  X |     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |  X |     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |    |     |
|  X |    |    |    |    |    |     |  O |    |    |    |    |    |     |
|  X |    |    |    |    |    |     |  O |    |    |    |    |    |     |
|-----------------------------|     |-----------------------------|     |
|  O |    |    |    |    |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |    |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |  O |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |  O |     |
|--------Outer Board----------|     |-------P=X Home Board--------|     |
| 11 | 10 |  9 |  8 |  7 |  6 | BAR |  5 |  4 |  3 |  2 |  1 |  0 | OFF |
```

### <a name="episode_termination"></a>エピソードの終了
1. 2人のプレーヤーのうち1人がゲームに勝つ
2. エピソードの長さが10000より大きい


### <a name="reset"></a>リセット
メソッド `reset()`:
- 最初に行動するプレーヤーを指定する (`WHITE`は`0`、`BLACK`は`1`) 
- サイコロの最初の出目、すなわち、`BLACK`の場合は`(1,3)`、`WHITE`の場合は`(-1,-3)`のように、振ったサイコロのタプルです。
- スタート位置からの観測機能


### <a name="rendering"></a>レンダリング
`render(mode = 'rgb_array')` や `render(mode = 'state_pixels')` が選択された場合、(複数のステップで)このような出力が得られます。
<p align="center">
   <img alt="Backgammon" title="Backgammon" src="./board.gif" width="450">
</p>

---
### <a name="example"></a>例
#### <a name="play"></a>Play Random Agents
簡単な例を挙げてみましょう。（両方のエージェント - `WHITE` と `BLACK` がランダムに行動を選択します）
```
cd examples/
python3 play_random_agent.py
```
#### <a name="valid_actions"></a>有効なアクション
内部変数の`current player`は、プレイヤーを順番に追跡するために使用されます。（プレイヤーの色を表します） 

有効なアクションをすべて取得するには、

```python
actions = env.get_valid_actions(roll)
```

有効なアクションは、タプルのセットとして表現されます。  
各アクションは、`((source, target), (source, target))`という形式のタプルである。  
各タプルは `(source, target)` という形式の動きを表します。  

#### NOTE:
「`double`を要求する」「`double`を受け入れる／拒否する」といったアクションはできません。 

次のような構成の場合 ([starting position](#starting_position)、`BLACK`のターン、`roll = (1, 3)`):
```
| 12 | 13 | 14 | 15 | 16 | 17 | BAR | 18 | 19 | 20 | 21 | 22 | 23 | OFF |
|--------Outer Board----------|     |-------P=O Home Board--------|     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |  X |     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |  X |     |
|  X |    |    |    |  O |    |     |  O |    |    |    |    |    |     |
|  X |    |    |    |    |    |     |  O |    |    |    |    |    |     |
|  X |    |    |    |    |    |     |  O |    |    |    |    |    |     |
|-----------------------------|     |-----------------------------|     |
|  O |    |    |    |    |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |    |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |    |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |  O |     |
|  O |    |    |    |  X |    |     |  X |    |    |    |    |  O |     |
|--------Outer Board----------|     |-------P=X Home Board--------|     |
| 11 | 10 |  9 |  8 |  7 |  6 | BAR |  5 |  4 |  3 |  2 |  1 |  0 | OFF |

Current player=1 (O - Black) | Roll=(1, 3)
```
有効なアクションは、
```
Legal Actions:
((11, 14), (14, 15))
((0, 1), (11, 14))
((18, 19), (18, 21))
((11, 14), (18, 19))
((0, 1), (0, 3))
((0, 1), (16, 19))
((16, 17), (16, 19))
((18, 19), (19, 22))
((0, 1), (18, 21))
((16, 17), (18, 21))
((0, 3), (18, 19))
((16, 19), (18, 19))
((16, 19), (19, 20))
((0, 1), (1, 4))
((16, 17), (17, 20))
((0, 3), (16, 17))
((18, 21), (21, 22))
((0, 3), (3, 4))
((11, 14), (16, 17))
```
---

## <a name="versions"></a>Backgammon Versions 
### `backgammon-v0`
上記の説明は、`backgammon-v0`を参照しています。

### `backgammon-pixel-v0`
この状態は、特徴ベクトル`(96, 96, 3)`で表されます。  
これが、`backgammon-v0`との唯一の違いです。

ボードの表現方法の一例
<p align="center">
   <img alt="raw_pixel" title="raw_pixel" src="./raw_pixel.png" width="150">
</p>


---
## <a name="useful_links"></a>Useful links and related works
- [1][Implementation Details TD-Gammon](http://www.scholarpedia.org/article/User:Gerald_Tesauro/Proposed/Td-gammon)
- [2][Practical Issues in Temporal Difference Learning](https://papers.nips.cc/paper/465-practical-issues-in-temporal-difference-learning.pdf)
<br><br>   
- Rules of Backgammon:
    - www.bkgm.com/rules.html
    - https://en.wikipedia.org/wiki/Backgammon
    - <a name="starting_position"></a>Starting Position: http://www.bkgm.com/gloss/lookup.cgi?starting+position
    - https://bkgm.com/faq/
<br><br>   
- Other Implementation of TD-Gammon and the game of Backgammon:
    - https://github.com/TobiasVogt/TD-Gammon
    - https://github.com/millerm/TD-Gammon
    - https://github.com/fomorians/td-gammon
<br><br>
- Other Implementation of the Backgammon OpenAI Gym Environment: 
    - https://github.com/edusta/gym-backgammon

---
## <a name="license"></a>License
[MIT](https://github.com/dellalibera/gym-backgammon/blob/master/LICENSE) 