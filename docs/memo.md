

    IO.inspect [DateTime.compare(DateTime.now!("Etc/UTC"), data["set_time"]), DateTime.now!("Etc/UTC"), data["set_time"]]
    with oversec when oversec eq :gt <- DateTime.compare(DateTime.now!("Etc/UTC"), data["set_time"]) do
    else
      _ -> nil # nothing to do
未整理


Q. ハッシュ化

`:crypto.hash(:sha256, data) |> Base.encode16(case: :lower)`

https://qiita.com/astap/items/ea1bd45ea54dc0183290