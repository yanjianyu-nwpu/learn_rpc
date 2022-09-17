# Intro

## 0 writer.go

### 0.1 writer

    结构体

```
// Writer writes data to connection
type Writer struct {
	ctx     context.Context
	conn    net.Conn
	timeout int
}
```

    context 不用多说，conn是网络链接


