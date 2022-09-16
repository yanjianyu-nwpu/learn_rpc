# Channel

## 0 ch简介

非常简单，interface

```
// Transport for initiate a new stream
    Transport interface {
        Pipe() (Sender, Receiver, error)
    }

    // Sender for send message
    Sender interface {
        Send(ctx context.Context, cmd qrpc.Cmd, msg interface{}, end bool) error
        // signal for the end of stream
        End() error
    }

    // Receiver for receive message
    Receiver interface {
        Receive(ctx context.Context, cmd *qrpc.Cmd, msg interface{}) error
    }
```

    这里transport，是用于初始化 sender 和receiver

## 1 encode.go 简介

    简单的json解码编码



## 2 clienttransport.go


