# PBWM

[![GoDoc](https://godoc.org/github.com/emer/leabra/pbwm?status.svg)](https://godoc.org/github.com/emer/leabra/pbwm)

# タイミング

## クオーターとロンガー

`GateState.Cnt`は、ゲーティング状態のキートラッカーを提供しています。

- -1 =この値に初期化されます。維持されるわけではありません
- 0 =ゲートされた値です。thal（視床？）の活動値がゲーティングの閾値を超えるたびに、カウンタをリセット（リゲート）します
- > = 1：保持。最初のゲーティングは、ゲーティングクオーター直後のQuarter_Initで1になり、その後カウントアップします。
- <= -1：保持しない。クリアされると、クリアするクオーター直後のQuarter_Initで-1にリセットされ、その後カウントダウンします。

Trial.Qtr | Phase | BG | PFCmnt | PFCmntD | PFCout | PFCoutD | Notes                       a
--- | --- | --- | --- | --- | --- | --- | ---
1.1 | init |  |  |  |  |  | 
1.1 | -- | GPi Go -> PFCmnt.thal | input -> act | Ctxt = 0, no act | input -> act | Ctxt = 0, no act | cortico-cortical super only, gate mid-quarter
1.2 | -- |  | Burst = (act > thr) * thal_eff | " | " | " | mnt super d5b gets gating signal
1.3 | init |  | thal > 0: ThalCnt++ | s Burst -> Ctxt, Ctxt > 0: ThalCnt++ |  | " | mnt deep gets d5b gated "context"
1.3 | -- |  | DeepLrn + input -> act | Ctxt -> act -> s DeepLrn; out?; trc | gets mnt deep input (opt) | " | mnt continues w/ no out gating, out gets primed
1.4 | + | Other Go | Burst = (act > thr) * thal_eff | trc d5b clamp -> netin, learning | " | " | TRC auto-encoder learning for mnt, via deep
2.1 | init |  | ThalCnt > 0: ThalCnt++ | s Burst -> Ctxt, Ctxt > 0: ThalCnt++ |  | " | mnt continues
2.1 | -- |  | " | " | " | " | 
2.2 | -- | GPi Go -> PFCout.thal | " | " | Burst = (act > thr) * thal_eff | " | out super d5b gets gating signal
2.3 | init |  | ThalCnt < 0: cleared fm out, Burst = 0 | Ctxt = 0, ThalCnt = -1 | thal > 0: ThalCnt++ | s Burst -> Ctxt, Ctxt > 0: ThalCnt++ | out deep gets d5b gating
2.3 | -- |  | input -> act | Ctxt = 0, no act | DeepLrn + input -> act | Ctxt -> act -> s DeepLrn; output; trc | out gating takes effect, driving actual output
2.4 | + | Other Go | " | " | " | " | continued output driving
3.0 | init |  | ThalCnt < 0: ThalCnt-- | ThalCnt < 0: ThalCnt-- | ThalCnt > out_mnt: ThalCnt = -1, Burst = 0 | Ctxt = 0: ThalCnt = -1 | out gating cleared automatically after 1 trial

## サイクル

### C++ 版

- Gating Cyc:

    - Cycle() -- 通常
    - GateSend(): GPiThalLayer: ゲーティングを検出し、 *次のサイクル*に応答するすべての人にGateState信号を送信します。

- Cyc+1:

    - ComputeAct:
        - Matrix: PatchShunt, SaveGatingThal, OutAChInhib in ApplyInhib
        - PFC:    PFCGating

# TODO

- [ ] Matrix がnet_gain = 0.5を使用する。謎？？

- [] PFCDeepにはdynsやctrなどがありません。実際には機能していません

- [] PrjnsのLearnQtr：deepには現在これもない？

- [ ] Trace matrix learning と他のprjns
