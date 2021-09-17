# 性能評価

測定結果の可視化方法を説明します。

```
$ cd ~/ros2_ws/evaluate
$ jupyter-lab
```

本ページのサンプル jupyter notebook については以下をご覧ください。
[https://github.com/tier4/CARET_demos/blob/main/samples/end_to_end_sample/visualize_result.ipynb](https://github.com/tier4/CARET_demos/blob/main/samples/end_to_end_sample/visualize_result.ipynb)

本ページのコマンドを実行する前に、以下を実行する必要が有ります。
```
from bokeh.plotting import output_notebook, figure, show
output_notebook()

import caret_analyze as caret
import caret_analyze.plot as caret_plot
```

測定結果の読み込み、測定対象のパスの指定配下のようにして行います。
```
# トレース結果の読み込み、 キャッシュを使わない場合は force_conversion=True とする。
lttng = caret.Lttng('end_to_end_sample', force_conversion=True)
# アーキテクチャファイルを読み込み、トレース結果を紐付ける
app = caret.Application('architecture.yaml', 'yaml', lttng)
# 評価対象のパス名を指定。
path = app.path['target_path']
```


## メッセージフロー

```python
# トレース結果の読み込み、 キャッシュを使わない場合は force_conversion=True とする。
caret_plot.message_flow(path, granularity='node', treat_drop_as_delay=True)
```

`treat_drop_as_delay=False`とした場合、メッセージがロスト（上書き）されたトレースポイントの箇所で途切れます。
`treat_drop_as_delay=True`とした場合、ロスト箇所を遅延として算出できるように紐付けたフローを出力します。

![message_flow_sample](/imgs/message_flow_sample.png)

## チェーンのレイテンシ

```python
caret_plot.chain_latency(path, granularity='node', treat_drop_as_delay=True)
```

`treat_drop_as_delay=False`とした場合、ロストせず終点まで到達したメッセージのレイテンシを出力します。
`treat_drop_as_delay=True`とした場合、ロスト箇所を遅延として算出したメッセージのレイテンシを出力します。

![chain_latency_sample](/imgs/chain_latency_sample.png)

## レイテンシの時系列波形・ヒストグラム

```python
t, latency_ns = path.to_timeseries(remove_dropped=False, treat_drop_as_delay=True)
latency_ms = latency_ns * 1.0e-6

p = figure()
p.line(t, latency_ns)
show(p)
```

![time_series_sample](/imgs/time_series_sample.png)


ヒストグラムも以下のようにして可視化できます。

```python
bins, hist = path.to_histogram(treat_drop_as_delay=True)
p = figure()
p.step(hist[1:], bins)
show(p)
```

![history_sample](/imgs/history_sample.png)

時系列とヒストグラムについては、以下が指定可能です。

```python
callback = app.callbacks[0] # コールバック実行時間
node = app.nodes[0].paths[0] # ノードレイテンシ
path = app.paths[0] # パスレイテンシ
comm = app.communications[0] # 通信レイテンシ（pub_sub レイテンシ）
comm_dds = app.communications[0].to_dds_latency() # 通信レイテンシ（DDSレイヤーレイテンシ）
var_pass = app.variable_passings[0] # 変数を介したメッセージ渡し
```