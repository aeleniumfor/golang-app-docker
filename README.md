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
ちなみにディレクトリ構成は以下のようになっている

## 今までのやり方
比較のため今までのdocker fileを紹介する

before/dockerfile

```dockerfile
FROM golang:1.10-alpine3.7
ADD ./ ./
RUN go build main.go
CMD [ "/go/main" ]
```
これを`docker build`してみる

```bash
$ docker build -t golang-docker-before before/

Successfully tagged golang-docker-before:latest
```
これが出てきたらbuildは正常に終わっているだろう
次に`イメージのサイズを見る

```bash
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
golang-docker-before   latest              b289ef900ec0        3 minutes ago       264MB
golang                 1.10-alpine3.7      7f9031684cc1        3 days ago          257MB

```
さっき作ったgolagn-docker-beforeのイメージサイズはどうだろうか?  
264Mある....でかい  
そもそもgolangのイメージが大きい  
