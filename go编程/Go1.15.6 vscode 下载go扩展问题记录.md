##### 在下载扩展的时候出现

```
Installing github.com/mdempsky/gocode FAILED
Installing github.com/ramya-rao-a/go-outline FAILED
Installing github.com/acroca/go-symbols FAILED
Installing golang.org/x/tools/cmd/guru FAILED
Installing golang.org/x/tools/cmd/gorename FAILED
Installing github.com/stamblerre/gocode FAILED
Installing github.com/ianthehat/godef FAILED
Installing github.com/sqs/goreturns FAILED
```

解决方案

```
go env -w GOPROXY=http://goproxy.io,direct
```

设置代理，原来的代理我们无法访问，被墙

