# Stream

## 0 Intro

    stream 流式传输

```
// ConnStreams hosts all streams on connection
// 感觉这个是控制所有的stream
type ConnStreams struct {
    streams     sync.Map // map[uint64]*Stream
    pushstreams sync.Map // map[uint64]*Stream
}
```

记录素有stream

## 1 getStream

```
// GetStream tries to get the associated stream, called by framereader for rst frame
// 尝试进入 framereader用来读取 rst frame
func (cs *ConnStreams) GetStream(requestID uint64, flags FrameFlag) *Stream {

    var target *sync.Map
    if flags.IsPush() {
        target = &cs.pushstreams
    } else {
        target = &cs.streams
    }

    v, ok := target.Load(requestID)
    if !ok {
        return nil
    }
    return v.(*Stream)
}
```

通过requestID和 falgs 得到steam的指针



```
// CreateOrGetStream get or create the associated stream atomically
// if PushFlag is set, should only call CreateOrGetStream if caller is framereader
func (cs *ConnStreams) CreateOrGetStream(ctx context.Context, requestID uint64, flags FrameFlag) (*Stream, bool) {

	var target *sync.Map
	if flags.IsPush() {
		target = &cs.pushstreams
	} else {
		target = &cs.streams
	}

	s := newStream(ctx, requestID, func() {
		l.Debug("close stream", zap.Uintptr("cs", uintptr(unsafe.Pointer(cs))), zap.Uint64("requestID", requestID), zap.Uint8("flags", uint8(flags)))
		target.Delete(requestID)
	})
	v, loaded := target.LoadOrStore(requestID, s)
	if loaded {
		// otherwise ml happens
		// TODO refactor stream!
		s.reset()
	}
	return v.(*Stream), loaded

}
```

如果没有创建一个

```

```

## 2 Stream 结构体
