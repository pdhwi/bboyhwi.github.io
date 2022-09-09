# go-Dockerfile


## 尝试构建项目并部署，记录一下。

1.构建一个编译go的镜像，生成go项目的可执行文件

2.构建一个运行镜像，把可执行文件、配置文件等相关文件放入镜像，运行可执行文件。 
<!--more-->


## 完整的Dockerfile 

```markdown
FROM golang:alpine AS builder

LABEL stage=gobuilder

ENV CGO_ENABLED 0
ENV GOPROXY https://goproxy.cn,direct

RUN apk update --no-cache && apk add --no-cache tzdata

WORKDIR /build

ADD go.mod .
ADD go.sum .
RUN go mod download
COPY . .
RUN go build -ldflags="-s -w" -o /app/go-api ./main.go


FROM alpine

RUN apk update --no-cache && apk add --no-cache ca-certificates
COPY --from=builder /usr/share/zoneinfo/Asia/Shanghai /usr/share/zoneinfo/Asia/Shanghai
ENV TZ Asia/Shanghai

WORKDIR /app
COPY ./config /app/config
COPY --from=builder /app/go-api /app/go-api

EXPOSE 8000
CMD ["./go-api"]
```

## 说明
代码块中 `FROM golang:alpine AS builder ..... ` 用于编译

代码块中 `FROM alpine ..... ` 使用编译后的go文件打包成镜像并暴露8000端口


## 运行
1.Dockerfile  放入项目根目录

2.项目根目录执行 docker build -t 自定义名称:版本号(go-api:1.0.0) . 

3.docker run -d --name=go-api  -p 8000:8000   自定义名称:版本号(go-api:1.0.0)


**结束啦**

访问浏览器 http://localhost:8000/ 查看，可执行 docker logs go-api 查看运行是否出问题
