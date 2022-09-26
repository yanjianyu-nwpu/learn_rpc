# frame

## 0 背景

## 1 FrameFlage

```
const (
	// 是否是stream
	// StreamFlag means packet is streamed
	StreamFlag FrameFlag = 1 << iota
	// 是不是结束了
	// StreamEndFlag denotes the end of a stream
	StreamEndFlag
	// StreamRstFlag 标志在取消stream 或者 显示出现了error的时候发送
	// StreamRstFlag is sent to request cancellation of a stream or to indicate that an error condition has occurred
	StreamRstFlag
	// NBFlag 意味着必须非阻塞的处理，用于stream fram
	// NBFlag means it should be handled nonblockingly, it's implied for streamed frames
	NBFlag
	// 是否是push
	// PushFlag mean the frame is pushed from server
	PushFlag
	// 是否编码··	q
	// CodecFlag for codec
	CodecFlag
)
```

      如上



## 2 frame结构体

```
// Frame models a qrpc frame
// all fields are readly only 所有字段都只能读
type Frame struct {
	RequestID uint64
	Flags     FrameFlag
	Cmd       Cmd
	Payload   []byte
	Stream    *Stream     // non nil for the first frame in stream 如果是整个stream第一个frame不位nil
	stash     interface{} // store per frame stuff, mainly used by middleware 存储所有frame
}
```
