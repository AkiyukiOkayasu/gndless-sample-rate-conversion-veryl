# Changelog

## [Unreleased]

### Changed

- 破壊的変更: `LinearAsrc`、`ContinuousLinearAsrc`、`CubicLagrangeAsrc`、`TwoXHalfbandAsrc`、`FourXHalfbandAsrc`を`FORMAT` genericと`FixedPointValue::<FORMAT>` value interfaceへ移行し、既定formatをQ2.23へ変更
- 内部FIFO・補間windowは`FORMAT::Raw`で保持し、public valueのformat境界だけをinterfaceで検査する構成へ整理
- fixed-rate benchmarkは既存の32bit raw条件を維持しつつ、halfband ASRC経路へQ1.31 interfaceを明示するよう更新

## [0.2.0] - 2026-08-02

### Added

- 汎用`PhaseIncrementEstimator`を追加し、sample-rate trackingから連続ASRC用の位相増分を生成可能にした
- `TwoXHalfbandAsrc`（set1、2倍halfband＋連続線形ASRC）を追加。set2候補は20kHz帯域の減衰とスペクトルイメージ成分の抑制が悪化するため既定経路には採用しない

### Changed

- スペクトルイメージ成分の解析説明を明確化

## [0.1.0] - 2026-08-02

- 公開moduleのparam/port doc commentを追加し、説明文の途中改行を整理
- doc commentの句点と体言止めの表記を整理
- doc commentのsummary表記を統一
- 各testのdoc commentを検証目的が分かる表現へ統一

### Changed

- 固定レートASRC benchmarkの入力を48点の固定正弦波vectorへ変更し、`oscillator`依存を削除
- 固定レートASRC benchmarkと解析スクリプトをpackageへ移動
- package名を`asrc`から`sample_rate_conversion`へ変更し、同期型sample-rate converterも収容できる構成へ整理
- ASRC本体を独立packageへ移動
- `FarrowAsrc`を`CubicLagrangeAsrc`へ改名
- 補間、phase accumulator、halfbandを依存packageへ分離
- 補間kernelのdefault nearest ties to even変更に合わせ、cubic統合goldenを更新
- Linear ASRCの起動・窓補充・underflowと4倍HBF経路のburstを示すWavedromを追加
- `SampleRateTracker`の測定開始、`period_valid`、lock獲得を示すWavedromを追加
