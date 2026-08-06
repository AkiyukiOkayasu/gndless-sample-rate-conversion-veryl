# gndless_sample_rate_conversion

同期・非同期のsample-rate converterのRTL実装。開発の極めて初期段階です。

- 任意比のASRC: `LinearAsrc`、`ContinuousLinearAsrc`、`CubicLagrangeAsrc`。入力の受理は`input_valid && input_ready`、出力はmoduleごとのoutput tickまたは連続clockで進みます。ratioは整数3bitを含むQ形式、FIFO depthとstartup levelはparameter、underflowはstickyです。
- 固定比のupsampler: `HalfbandUpsampler`。複数channelを1個のMACでTDM処理する2x halfbandで、係数は`HalfbandCoefficientSet` package（既定`Halfband103Q1_17`）から供給します。
- レート計測: `SampleRateTracker`、`PhaseIncrementEstimator`。

sample portは`FixedPointValue::<FORMAT>`で指定し、ASRC群の既定formatはQ2.23です。`HalfbandUpsampler`はQ3.24固定です。ADATのQ1.23 PCMを接続する場合は、境界に`FormatConverter`を置いてQ2.23へ変換します。

`PhaseIncrementEstimator`は非同期入力の`sample_valid`到着周期から、`gndless_nco::FractionalPhaseAccumulator`へ渡すQ0位相増分を生成します。通常の到着ジッターをIIRで平滑化しつつ、大きなレート変更、入力timeout、除算中の更新、位相増分が1.0以上となる範囲外入力を安全に扱います。

```veryl
inst asrc: sample_rate_conversion::CubicLagrangeAsrc (...);
```
