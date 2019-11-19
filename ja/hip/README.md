# 海馬

[![GoDoc](https://godoc.org/github.com/emer/leabra/hip?status.svg)](https://godoc.org/github.com/emer/leabra/hip)

パッケージhipは、シータ周波数を実装するための特別な海馬アルゴリズムを提供しています。
hippocampus model from Ketz, Morkonda, & O'Reilly (2013).

ThetaPhaseダイナミクスのタイミング，クオーター構造に基づいて：

- **Q1（第1クオーター）:**   ECin -> CA1 -> ECout (CA3 -> CA1 off)  : ActQ1 = auto-encoder用のマイナスフェーズ
- **Q2,3（第2，第3クオーター）:** CA3 -> CA1 -> ECout  (ECin -> CA1 off) : ActM = 再生用のマイナスフェーズ
- **Q4（第4クオーター）:**   ECin -> CA1, ECin -> ECout (CA3 -> CA1 off, ECin -> CA1 on): ActP = プラスフェーズ

```
[  q1      ][  q2  q3  ][     q4     ]
[ ------ minus ------- ][ -- plus -- ]
[   auto-  ][ recall-  ][ -- plus -- ]

  DG -> CA3 -> CA1
 /    /      /    \
[----ECin---] -> [ ECout ]

minus phase: ECout unclamped, driven by CA1
auto-   CA3 -> CA1 = 0, ECin -> CA1 = 1
recall- CA3 -> CA1 = 1, ECin -> CA1 = 0

plus phase: ECin -> ECout auto clamped
CA3 -> CA1 = 0, ECin -> CA1 = 1
(same as auto- -- training signal for CA3 -> CA1 is what EC would produce!
```

- `ActQ1` =auto encoderのマイナスフェーズ状態（EcCa1Prjnにおいて，CA1とECoutの両方で，CHLのActPプラスフェーズに対するマイナスフェーズとして使用される）。
- `ActM` = マイナスフェーズを呼び出します (CA3について通常のマイナスフェーズダイナミクス が学習を呼び出します)。
- `ActP` = プラス（autoとリコールの両方でプラスフェーズとして受け取られます）

学習は通常どおりトライアルの最後に行われますが、エンコーダの投射は，ActQ1、ActM、ActP変数を使用して，適切な信号で学習を行います。

todo：2試行バージョンのコードを実装して、真のシータリズムを生成します。
隣接する2つのアルファトライアルを統合します。
