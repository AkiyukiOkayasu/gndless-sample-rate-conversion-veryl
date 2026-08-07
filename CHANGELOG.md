# Changelog

## [0.4.0] - 2026-08-07

### Added

- `HalfbandUpsampler`（複数channelを1個のMACでTDM処理する2x halfband upsampler）を追加。既定係数setは`Halfband103Q1_17`
- `BurstFifo`（2 sample burstを吸収するdepth-2のvalid/ready FIFO）を追加。`HalfbandUpsampler`の2クロックburst出力を間欠的な入力readyの次段へ引き渡すために使う
- 4ch・2段cascade・burst入力(S/MUX相当)・BurstFifo・LinearAsrcのNative Testを追加

### Changed

- `HalfbandUpsampler`のhistoryをFFシフト配列からBSRAM推論対応のinline配列循環バッファ(2コピー)へ変更した。登録アドレス+登録出力を同一always_ffで行いGowinのBSRAM推論に適合させる。SRAM推論のためhistoryはresetを持たず、最初のHISTORY_LENGTH frame分の出力は不定値を含む。MACは同期読み出し(2clkレイテンシ)をprefetchパイプラインで吸収し、1 pair/cycleを維持する
- `LinearAsrc`の線形補間を3段パイプライン化した。乗算パスを分割してFmax 50MHz超を確保し、出力は窓確定から3clk後に現れる。丸め方式は変更なし
- Gowin合成のprocedural for制約(interface配列の定数選択不可)に対応し、modport配列をgenerate assignでplain配列へコピーして使う
- `Veryl.toml`に`clock_type`/`reset_type`を明示し、stdモジュールのリセット極性を利用側(sync_high)と一致させた
- `gndless_fixedpoint`依存を公開済みの0.2.2へ更新

## BREAKING CHANGE

- `ContinuousLinearAsrc`を`LinearAsrc`へ改名し、`FORMAT` genericと`FixedPointValue::<FORMAT>` value interfaceへ移行した。静的`ratio`を内部積算するオープンループ構成の`LinearAsrc`/`CubicLagrangeAsrc`は削除し、非同期経路を一本化した。外部フリーランニングクロック由来の非同期入力を`PhaseIncrementEstimator`+`FractionalPhaseAccumulator`が計測・追従する
- module境界のformatをQ4.23で統一した。`HalfbandUpsampler`のI/OをQ3.24から、`LinearAsrc`の既定をQ2.23から、それぞれQ4.23へ変更。±1.0の信号を±8.0の範囲で扱い、内部演算は境界formatを拡張しない
- `TwoXHalfbandAsrc`、`FourXHalfbandAsrc`を削除し、`HalfbandUpsampler`へ置き換え

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
