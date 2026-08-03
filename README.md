# gndless_sample_rate_conversion

同期・非同期のsample-rate converterのRTL実装。開発の極めて初期段階です。

入力の受理は`input_valid && input_ready`、出力はmoduleごとのoutput tickまたは連続clockで進みます。ratioは整数3bitを含むQ形式、FIFO depthとstartup levelはparameter、underflowはstickyです。

sample portは`FixedPointPort::<FORMAT>`で指定し、`LinearAsrc`、`ContinuousLinearAsrc`、Cubic、2x/4x halfband ASRCの既定formatはQ2.23です。ADATのQ1.23 PCMを接続する場合は、境界に`FixedPointSampleConverter`を置いてQ2.23へ変換します。

`PhaseIncrementEstimator`は非同期入力の`sample_valid`到着周期から、`gndless_nco::FractionalPhaseAccumulator`へ渡すQ0位相増分を生成します。通常の到着ジッターをIIRで平滑化しつつ、大きなレート変更、入力timeout、除算中の更新、位相増分が1.0以上となる範囲外入力を安全に扱います。

```veryl
inst asrc: sample_rate_conversion::CubicLagrangeAsrc (...);
```

## テスト

固定レートの実レート比比較はignored testでCSVを生成し、標準Pythonの解析スクリプトで直接線形補間と4倍HBF経路の振幅を比較します。

```text
veryl test --ignored -t fixed_rate_asrc_benchmark
python3 tools/analyze_fixed_rate_asrc.py target/fixed_rate_asrc_benchmark.csv
```
