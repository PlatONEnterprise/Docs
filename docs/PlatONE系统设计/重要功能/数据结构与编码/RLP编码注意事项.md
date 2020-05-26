[TOC]

# 1. RLP编码注意事项

## 1.1. 不支持的类型

* 非空的interface类型。例如：

  ```bash
  type Reader interface{
    Read()
  }
  ```

* bool类型

* 有符号整型

* 浮点类型

* map

* channel

* 函数

## 1.2. 自定义编解码方法

* 可以通过自定义DecodeRLP，EncodeRLP两个方法来解决以下问题
  * 不可导出的字段不会编码。
  * 非空interface类型
  * map类型

示例代码如下所示：

```go
type Proposal interface {
	Number() *big.Int
}

type Preprepare struct {
	View      *View
	
	//Proposal是一个非空ingterface类型，如果需要编解码此字段，需要自定义下EncodeRLP，DecodeRLP方法
	Proposal  Proposal
	
	//小写字母开头，不可以导出，如果需要编解码此字段，需要自定义下EncodeRLP，DecodeRLP方法
	lockRound *big.Int 
  messages   map[common.Address]*message
}

func (b *Preprepare) EncodeRLP(w io.Writer) error {
  //手动把哪些需要编码的对象一一填写进去，这样就可以把非导出字段进行编码，
  //也可以很灵活的控制只对哪些字段进行编码
	return rlp.Encode(w, []interface{}{b.View, b.Proposal, b.lockRound， b.Values(})
}

func (b *Preprepare) DecodeRLP(s *rlp.Stream) error {
  //注意字段的顺序必要和EncodeRLP方法中的字段顺序一样
	var preprepare struct {
		View      *View
		Proposal  *types.Block //必须定义为Proposal接口所指向的真正的类型
		LockRound *big.Int
    Messages []*message
	}

	if err := s.Decode(&preprepare); err != nil {
		return err
	}
	b.View, b.Proposal = preprepare.View, preprepare.Proposal

  for _, val := range preprepare.Messages {
		b.messages[val.Address] = val
	}
	return nil
}
                    
func (b *Preprepare) Values() (result []*message) {
	for _, v := range b.messages {
		result = append(result, v)
	}

	return result
}
```
