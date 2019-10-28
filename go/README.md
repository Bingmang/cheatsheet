# Go

## 常用指令

Go Mod 使用
```bash
go mod init     # 初始化（当项目不在GOPATH下时需要在后面加上package名字
go mod download # 下载所有依赖
go mod tidy     # 移除不需要的依赖包
go build        # 构建项目并自动分析依赖包，写入go.mod

```

## 常用包

Package   | Link
-|-
gin       | github.com/gin-gonic/gin
govendor  | github.com/kardianos/govendor
gorm      | github.com/jinzhu/gorm
postgresql| github.com/lib/pq
