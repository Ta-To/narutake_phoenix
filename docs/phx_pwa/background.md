バックグラウンド処理に関するノート

## 処理予約

## ポーリング方式

- 最初から繰り返し実行されるタスクを作成しておく
- 各プロセスに対して毎回実行するかどうかの判定を行わせる

``` elixir:例
defmodule TimeKeeper do
  def polling do
    task = Task.async(fn ->
      __MODULE__.check()
    end)
    :timer.sleep(1000) # interval
    Task.await(task)
    Task.shutdonw(task)
    __MODULE__.polling()
  end

  def check do
    # チェック対象のプロセスIDリストをもっておき、それぞれのプロセスに通知
  end
end
```

##

- https://sipsandbits.com/2020/08/07/do-we-need-background-jobs-in-elixir/
- https://medium.com/@cschneid/background-jobs-in-elixir-phoenix-60dddf4ce207
