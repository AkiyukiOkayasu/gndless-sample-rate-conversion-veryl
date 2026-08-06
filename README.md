# gndless_sample_rate_conversion

同期・非同期のsample-rate converterのRTL実装。開発の極めて初期段階です。

- 任意比のASRC: `LinearAsrc`。位相`phase`/`advance`を外部から受け、`PhaseIncrementEstimator`と`gndless_nco::FractionalPhaseAccumulator`で入力レートへ追従させ、連続clockで補間値を出力します。入力は`input_valid && input_ready`で受理し、underflowはstickyです。
- 固定比のupsampler: `HalfbandUpsampler`。複数channelを1個のMACでTDM処理する2x halfbandで、係数は`HalfbandCoefficientSet` package（既定`Halfband103Q1_17`）から供給します。
- レート計測: `SampleRateTracker`、`PhaseIncrementEstimator`。

sample portは`FixedPointValue::<FORMAT>`で指定し、module境界の既定formatはQ4.23で統一しています。±1.0の信号を±8.0の範囲で扱い、内部演算は境界formatを拡張しません。ADATのQ1.23 PCMを接続する場合は、境界に`FormatConverter`を置いてQ4.23へ変換します(fracが一致するため符号拡張のみ)。DSM(Q1.31)へは左シフトで無損失、I2S/AES3(Q1.23)へは飽和のみで変換します。

`PhaseIncrementEstimator`は非同期入力の`sample_valid`到着周期から、`gndless_nco::FractionalPhaseAccumulator`へ渡すQ0位相増分を生成します。通常の到着ジッターをIIRで平滑化しつつ、大きなレート変更、入力timeout、除算中の更新、位相増分が1.0以上となる範囲外入力を安全に扱います。

```veryl
inst asrc: sample_rate_conversion::LinearAsrc (...);
```
