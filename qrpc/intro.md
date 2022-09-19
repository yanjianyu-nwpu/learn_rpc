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

   

writer 函数

```
// Write writes bytes
func (w *Writer) Write(bytes []byte) (int, error) {
    var (
        endTime      time.Time
        writeTimeout time.Duration
        offset       int
        n            int
        err          error
    )
    // 这里是这个writer设置的每次发送的时间限制
    timeout := w.timeout
    if timeout > 0 {
        endTime = time.Now().Add(time.Duration(timeout) * time.Second)
    }
    size := len(bytes)

    for {
        if timeout > 0 {
            // 计算还能有多少时间发送
            writeTimeout = endTime.Sub(time.Now())
            // 大于三秒就限制到3秒
            if writeTimeout > CtxCheckMaxInterval {
                writeTimeout = CtxCheckMaxInterval
            }
        } else {
            writeTimeout = CtxCheckMaxInterval
        }

        // 设置context 超时时间
        w.conn.SetWriteDeadline(time.Now().Add(writeTimeout))

        // 计算每次能写多少n 因为一个有上限，二个buffer不一定够
        n, err = w.conn.Write(bytes[offset:])
        // 后移动offset
        offset += n
        if err != nil {
            //如果超时
            if opError, ok := err.(*net.OpError); ok && opError.Timeout() {
                if timeout > 0 && time.Now().After(endTime) {
                    return offset, err
                }
            } else {
                return offset, err
            }
        }
        // 如果发送完了
        if offset >= size {
            return offset, nil
        }

        // 如果writer 这边的超时了 ctx 超时
        select {
        case <-w.ctx.Done():
            return offset, w.ctx.Err()
        default:
        }
    }

}
```

这里有几参数

endTime

writeTimeOut

offset 

n

err

首先 timeout = w.timeout 就是记录下这个writer 超时的时间

然后计算endtime 

然后统计 每次能write的大小，循环调用（因为一次传输有上限）

如果超时了，直接返回

## 1 stream.go

     所有的流都放在一个ConnStreams结构体里面，感觉就是每个请求对应到对应的链接，stream

```
// ConnStreams hosts all streams on connection
type ConnStreams struct {
    streams     sync.Map // map[uint64]*Stream
    pushstreams sync.Map // map[uint64]*Stream

}
```

    这个int64 是每个请求的req id。然后这里有并发问题。

    

    FlameFlag uint8 感觉就是简单的位权控制

    stream 类似于每个tcp链接

```
// Stream is like session within one requestID
type Stream struct {
	ID         uint64
	frameCh    chan *Frame // always not nil, closed when peer is done
	ctx        context.Context
	cancelFunc context.CancelFunc

	// 这里感觉就是标志符号，closedself感觉就是自己没了，closepeer双端都挂呢，fullClosed程序都关了？
	closedSelf  int32
	closedPeer  int32
	fullClosed  int32
	binded      bool
	fullCloseCh chan struct{}
	closeNotify func() // called when stream is fully closed
}
```



    这里每个stream有个id， 然后每个chan 接受frameCh，然后有个调用cancel

     

    这里的blind 是 stream 有没有和fame 绑定

```
// TryBind returns true if the stream has not been binded to any frame yet
// streamreader will first call TryBind, and if fail, call AddInFrame
// not ts
// 就是尝试frame绑定stream
func (s *Stream) TryBind(firstFrame *Frame) bool {

	if s.binded {
		return false
	}
	s.binded = true

	s.closePeerIfNeeded(firstFrame.Flags)

	firstFrame.Stream = s

	return true
}
```

    这里这个 closePeerIfNeeded ，然后如果是IsPush frame那么会closedSelf                                                                                                                                                                                                                                                                                                                                                                                                                                          

    

## 2 frame.go

     这个frame，感觉是每个req 会有多个frame

## 3 qrpc.go

    IsPushed 是否是要推送的








