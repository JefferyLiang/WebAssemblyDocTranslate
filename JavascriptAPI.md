# JavaScript API

在[MVP]()里，通过JS API下明确定义的`WebAssembly`。（实际上`WebAssembly`也能被HTML的`<script type='module'>`标签直接的加载和运行--和其他的通过ES6模块的URL引入的Web API一样--作为ES6模块集成的一部分）

WebAssembly JS API的typescript定义文档可以在[这里](https://github.com/01alchemist/webassembly-types/blob/master/webassembly.d.ts)找到，启动它能够让WebAssembly对Typescirpt编写的代码完成自动的编译。

# Traps(语法错误？)

每当Webassembly的语法出现错误，`WebAssembly.RuntimeError`对象会将错误抛出。Webassembly的代码(当前的)没有办法捕获这些错误，并且将错误报告到外部环境(如：浏览器 或 Javascript代码中)。

如果WebAssembly引入了Javascript的错误捕获方法，那么错误将传播到引入的捕获者上。

因为JavaScript所以exceptions可以被处理，而且当Javascript处理了丢出的错误以后，能够继续运行WebAssembly，错误被处理，程序也能继续运行（这里不太清楚是不是应该这样子翻译）。

# 栈溢出

每当在 WebAssembly 的代码中发生栈溢出时，都会有一些栈溢出的警报抛出到Javascript里。

# WebAssembly 对象

WebAssembly对象是在global对象上拥有初始化值的对象。跟`Math`和`JSON`对象一样，`WebAssembly`对象是一个普通的JS对象(没有构造函数或者方法)就像一个命名空间和具有以下的属性:


## `WebAssembly [ @@toStringTag ]` Property

`@@toStringTag` 一个初始化的字符串属性。

这个属性具有一下的可配置属性:

```
{
  [[Writable]]: false,
  [[Enumerable]]: false,
  [[Configurable]]: true
}
```

## `WebAssembly`对象的构造属性

随着`WebAssembly`的构造，下这些属性将会被添加到对象之中:

* `WebAssembly.Module`: [`WebAssembly.Module` constructor](#Module)
* `WebAssembly.Instance`: [`WebAssebly.Instance` constructor](#Instance)
* `WebAssembly.Memory`: [`WebAssembly.Memory` constructor](#Memory)
* `WebAssembly.Table`: [`WebAssembly.Table` constructor](#Table)
* `WebAssembly.CompileError`: 这是一个[NavtiveError](),当`WebAssembly`在被编译和语法检测中出现的错误，将把错误信息从这里抛出。
* `WebAssembly.LinkError`: 这是一个[NavtiveError](),当`WebAssembly`实例化模块时出现错误，将把错误信息从这里抛出。
* `WebAssembly.RuntimeError`: 这是一个[NavtiveError](),当`WebAssembly`在运行时，出现了意外的错误，将会把错误信息从这里抛出。

## `WebAssembly`的对象的方法属性

### WebAssembly.validate

`validate`方法的定义:

```
  Boolean validate(BufferSource bytes)
```

如果给予的`bytes`参数不是`BufferSource`,将会抛出类型错误。

除此之外，这个函数还会验证定义的[WebAssembly 规范](),当验证成功时，返回`true`,否则返回`false`


### WebAssembly.compile

`compile`方法的定义：

```
  Promise<WebAssembly.Module> compile(BufferSrouce bytes)
```

如果传递的参数不是`BufferSource`，那么`Promise`将会`rejected`类型错误信息。

除此之外，这个异步方法将会启动一个编译生成一个`WebAssembly.Module`作为构造`Webassembly.Module`的描述。当编译成功,`Promise`将会返回一个`WebAssembly.Module`的对象结果。当失败的时候，`Promise`将会`rejected`编译错误的信息并从`WebAssembly.CompileError`中抛出。

这个异步编译的逻辑是，当我们使用`compile`时，拷贝一份`BufferSource`状态的副本。随后返回编译完成的`BufferSource`,而且不回影响当前的其他编译任务。

实际上，这个方法可以被扩展成stream流数据参数，从而实现异步的流编译。


### WebAssembly.instantiate

这个方法是用于根据参数类型进行重载，如果重载以后的类型还是不匹配，怎返回`Proimse`的`rejected`,并在里面附带类型警报。

```
dictionary WebAssemblyInstantiatedSource {
  required WebAssembly.Module module;
  required WebAssembly.Instance instance;
};

Promise<WebAssemblyInstantiatedSource>
  instantiate(BufferSource bytes [, importObject])
```

如果传递的参数不是`BufferSource`，那么`Promise`将会`rejected`类型错误信息。


