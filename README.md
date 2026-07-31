# gndless_sample_rate_conversion

同期・非同期のsample-rate converterのRTL実装。開発の極めて初期段階です。

入力の受理は`input_valid && input_ready`、出力はmoduleごとのoutput tickまたは連続clockで進みます。ratioは整数3bitを含むQ形式、FIFO depthとstartup levelはparameter、underflowはstickyです。

```veryl
inst asrc: sample_rate_conversion::CubicLagrangeAsrc (...);
```

## テスト

固定レートの実レート比比較はignored testでCSVを生成し、標準Pythonの解析スクリプトで直接線形補間と4倍HBF経路の振幅を比較します。

```text
veryl test --ignored -t fixed_rate_asrc_benchmark
python3 tools/analyze_fixed_rate_asrc.py target/fixed_rate_asrc_benchmark.csv
```
