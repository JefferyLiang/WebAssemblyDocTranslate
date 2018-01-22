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

这个方法将启动一个异步的编辑`WebAssembly.Module`到`bytes`的说明文件用于构造`WebAssembly.Module`,然后根据队列中的`importObject`参数作为`WebAssembly.Instance`构造说明实例化`Module`。  在实例化任务运行之后和在下一步的任务进行前,可能会运行其他未指定的异步任务。

当成功重载以后，`Promise`将会被完成并且返回一个包含 `{ module, instance }`的对象，他们分别对应`WebAssembly.Module`和`WebAssembly.Instance`。这两个属性将会根据我们之前初始化的参数决定它是否可配置，可枚举，可编写。

如果重载失败，`Promise`将会根据错误的信息`rejected`一个关于`WebAssembly.CompileError`、`WebAssembly.LinkError`或者`WebAssembly.RuntimeError`的错误。

这个异步编译的逻辑是，当我们使用`compile`时，拷贝一份`BufferSource`状态的副本。随后返回编译完成的`BufferSource`,而且不回影响当前的其他编译任务。

```
  Promise<WebAssembly.Instance> instantiate(moduleObject [, importObject])
```

上面的描述适用于实例化的第一个参数是`WebAssembly.Module`的例子。

当这个异步方法根据`moduleObject`和`importObject`的描述去构造`WebAssembly.Instance`的实例。在实例化任务运行之后和在下一步的任务进行前,可能会运行其他未指定的异步任务。

当成功重载以后，`Promise`将会被完成并且返回一个包含 `{ module, instance }`的对象，他们分别对应`WebAssembly.Module`和`WebAssembly.Instance`。如果重载失败，`Promise`将会根据错误的信息`rejected`一个关于`WebAssembly.CompileError`、`WebAssembly.LinkError`或者`WebAssembly.RuntimeError`的错误。

### `WebAssembly.Module` Objects

`WebAssembly.Module`是编译后的二进制模块，并且包含一个内部的slot:

* [[Module]]: 根据我们实例化模块的定义信息

#### 构造`WebAssembly.Module`

`WebAssembly.Module`方法定义:

```
  new Module(BufferSource bytes)
```

如果沒有 new 标记是 `undefined`，会抛出一个类型错误警报。

如果参数`bytes`不是`BufferSource`，会抛出一个类型错误警报。

除此之外，这个方法会根据`BufferSource`同步进行编译:

1. 根据[BinaryEncoding.md]()的解码规则对`BufferSource`进行解码，并且验证是否符合[spec/valid.ml]()的规则。

2. 根据[Web.md]()中的描述，在`Ast.module`的Spec字符串会被解码成UTF8的编码格式。

3. 当成功的时候，新的`WebAssembly.Module`对象会被返回，并且被设置成认真的`Ast.module`。

4. 如果失败，则抛出一个编译错误警报。
