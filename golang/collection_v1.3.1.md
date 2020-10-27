# collection v1.3.1升级记录

项目地址： https://github.com/jianfengye/collection  欢迎star。

collection 手册地址： http://collection.funaio.cn/

collection库升级到v1.3.1版本。

从v1.2.0 到v1.3.1 开发做了如下改动：

* 说明文档改造成线上手册
* 增加了 ObjPointCollection 结构
* 增加了 toObjs 方法
* 重构了 AbsArray
* 增加了 ContainsCount 方法
* errors 库换成 github.com/pkg/errors 库


# 说明文档改造成线上手册 

这个是组内前端同学的启发，他们写文档手册喜欢用vuepress。于是我将之前的库说明手册，一个大文件 README.md，修改成为了 docs 目录下的“指南” 和 “手册” 两个部分。手册中每个方法都是一个 markdown。

我的思考是这样把方法的说明拆分开，后续手册中可以扩展一些使用示例等。使用 vuepress 的好处还是不少的，一个是在自己的阿里云服务器上搭建了一个手册地址：http://collection.funaio.cn/ 。 这个地址让我在开发过程中更方便查看了，不用再上 github 上面看了。毕竟 github 最近也不是那么好登陆了。

![20201022131102](http://tuchuang.funaio.cn/md/20201022131102.png)

这里可以安利下 vuepress，很好用的一个 markdown 转 html 的工具。

# 增加了 ObjPointCollection 结构

这个需求源自我业务开发中遇到的需求。指针对象数组。我们希望指针对象数组也能使用 Colleciton 库的所有方法。所以增加了这个方法。

```golang


type FooBar struct {
	Foo string
	Bar int
}

func FooBarCompare(a interface{}, b interface{}) int {
	aobj := a.(*FooBar)
	bobj := b.(*FooBar)
	return aobj.Bar - bobj.Bar
}

func InitFooObjPoints() []*FooBar {
	return []*FooBar{
		{
			Foo: "astring",
			Bar: 1,
		},
		{
			Foo: "bstring",
			Bar: 2,
		},
	}
}

func TestObjPointCollection_Normal(t *testing.T) {
	objs := InitFooObjPoints()
	coll := NewObjPointCollection(objs).SetCompare(FooBarCompare)

	// 	[Append](#Append) 挂载一个元素到当前Collection
	{
		count := coll.Copy().Append(&FooBar{
			Foo: "cstring",
			Bar: 3,
		}).Count()
		if count != 3 {
			t.Fatal("append error")
		}
	}

	// [Contain](#Contain) 判断一个元素是否在Collection中
	{
		obj := objs[0]
		if coll.Contains(obj) != true {
			t.Fatal("contains error")
		}
	}

	// [Copy](#Copy) 根据当前的数组，创造出一个同类型的数组
	{
		if coll.Copy().Count() != 2 {
			t.Fatal("copy error")
		}
    }
}

```

# 增加了 toObjs 方法

这个方法是针对 ObjCollection 和 ObjPointCollection 设计的。 如果我们想要将 Collection 还原成对象数组，或者对象指针数组的时候，可以使用这个方法。这个方法使用了反射。

```golang

func TestObjPointCollection_ToObjs(t *testing.T) {
	a1 := &Foo{A: "a1", B: 1}
	a2 := &Foo{A: "a2", B: 2}
	a3 := &Foo{A: "a3", B: 3}

	bArr := []*Foo{}
	objColl := NewObjPointCollection([]*Foo{a1, a2, a3})
	err := objColl.ToObjs(&bArr)
	if err != nil {
		t.Fatal(err)
	}
	if len(bArr) != 3 {
		t.Fatal("toObjs error len")
	}
	if bArr[1].A != "a2" {
		t.Fatal("toObjs error copy")
	}
}

```

# 重构了 AbsArray 

之前的这篇 http://collection.funaio.cn/guide/introduce.html 说了我当时设计 collection 库的思考。但是在1.3.1 版本的时候，觉得实现的思维还是不够清晰。这次我的改造包括在底层 AbsArray 中存储了上层 collection 的类型。

```
const (
	TYPE_UNKNWON EleType = iota
	Type_INT
	Type_INT64
	Type_INT32
	TYPE_STRING
	TYPE_FLOAT32
	TYPE_FLOAT64
	TYPE_OBJ
	TYPE_OBJ_POINT
)
```

然后在内部实现了 must 相关的防御方法: 
```
mustSetCompare
mustBeNumType
mustBeBaseType
mustNotBeBaseType
mustNotBeEmpty
```

最后在每个具体实现的方法前先进行防御判断。这样整体代码可读性会得到提升。

# 增加 ContainsCount 方法

这个方法也是使用过程中提到的，我们希望不仅仅判断一个元素是否在数组中，也想判断这个元素在数组中出现了几次。于是便有了这个方法。

```golang
func TestAbsCollection_ContainsCount(t *testing.T) {
	intColl := NewIntCollection([]int{1, 2, 2, 3})
	count := intColl.ContainsCount(2)
	if count != 2 {
		t.Fatal(errors.New("contains count error"))
	}
}

```

# errors 换成 github.com/pkg/errors

官方 errors 库换成 pkg/errors 库的好处这里就不赘述了，有兴趣的可以参考 https://www.bilibili.com/video/BV1hE411c7Ze/ 

# 总结

最近内部又有一个新的模块服务使用 collection 库进行业务开发，真实感受加快了不少开发速度。

祝用的高兴。
