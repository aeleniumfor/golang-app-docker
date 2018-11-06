# なぜやるの?
dockerでアプリを動かすときdocker imageのサイズが大きくなる
このimageサイズをできるだけ小さくしたい
# どうやるの?
今回はdocker fileがメインなのgoファイルは共通にしている
使用するプログラムは以下になる
main.go

```go
package main

import (
  "fmt"
  "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello, World")
}

func main() {
  http.HandleFunc("/", handler)
  http.ListenAndServe(":8080", nil)
}
```
簡単なwebサーバでこれを動かして`curl`を叩くと`Hello, World`が帰ってくる
