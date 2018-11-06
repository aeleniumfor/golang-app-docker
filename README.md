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


## 何が必要か
golangで作成したアプリには何が必要だろうか?  
golangでbuildすると一つのファイルが吐き出される  
つまりこれだけアレばいいはずである  
今回でいうとmain.goでもその他パッケージでもなくmainが必要である  

## イメージを小さくする
mainのファイルさえあればいいのでmainのファイルをbuild時に取り出す  
そこでdockerでマルチステージビルドを使う  
マルチステージビルドについては[docker docs](https://docs.docker.com/)を見るといいだろう  
docker fileは以下のようになる  

after/dockerfile  

```dockerfile
FROM golang:1.10-alpine3.7 as build
ADD ./ ./
RUN go build main.go

FROM alpine:latest
COPY --from=build /go/main /app/main
CMD [ "/app/main" ]
```
docker fileを見てみよう最初のFROMではイメージの横に`build`というのがついている.これは任意の文字をつけることができる  
golangのimageを使って`go build`を行っている  
吐き出されたmainはどこにあるかというと`/go/main`にあるのでそれだけを取り出す  
２つ目にFROMでイメージサイズが小さいalpineを使用する  
COPYで`--from=build`でどのimageからCOPYからコピーしてくる`build`は最初のFROMでつけたものを使う  
そして`/app/main`に持っていく  
最後にCMDを指定してあげて終了  

## 修正後のimageサイズは

```bash
$ docker build -t golang-docker-after after/
$ docker images

golang-docker-after    latest              c9e1ed0e2383        38 minutes ago      11MB
<none>                 <none>              fece0e20da1f        38 minutes ago      264MB
golang-docker-before   latest              b289ef900ec0        About an hour ago   264MB
golang                 1.10-alpine3.7      7f9031684cc1        3 days ago          257MB
alpine                 latest              196d12cf6ab1        7 weeks ago         4.41MB
```
golang-docker-afterとgolang-docker-beforeのイメージサイズを比べてみる  

結果は以下のようになる  
golang-docker-after:11MB  
golang-docker-before:264MB  

## まとめ
そもそもimageサイズを小さくするメリットは`docker pull`が速くなることだろう  
これはdocker pullを頻繁にやる環境では有効的だと考える.  

ありがとうございました

