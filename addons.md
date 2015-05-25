# C/C++ 插件

插件 Addons 是动态链接的共享对象。他提供了 C/C++ 类库能力。这些API比较复杂，他包以下几个类库:

 - V8 JavaScript,  C++ 类库。用来和 JavaScript 交互，比如创建对象，调用函数等等。
   在 `v8.h` 头文件中 (目录地址`deps/v8/include/v8.h`)，线上地址
   [online](http://izs.me/v8-docs/main.html)。

 - [libuv](https://github.com/joyent/libuv)，C 事件循环库。
   等待文件描述符变为可读，等待定时器，等待信号时，会和 libuv 打交道。或者说，如果你需要和 I/O 打交道，就会用到 libuv。

 - 内部 Node 类库。 其中最重要的类 `node::ObjectWrap`，你会经常派生自它。

 - 其他的参见 `deps/`。

Node 已经将所有的依赖编译成可以执行文件，所以你不必当心这些类库的链接问题。

以下所有例子可以在
[download](https://github.com/rvagg/node-addon-examples) 下载，也许你可以从中找一个作为你的扩展插件。

## Hello world

现在我们来写一个 C++ 插件的小例子，它的效果和以下 JS 代码一致：

    module.exports.hello = function() { return 'world'; };

创建 `hello.cc` 文件:

    // hello.cc
    #include <node.h>

    using namespace v8;

    void Method(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world"));
    }

    void init(Handle<Object> exports) {
      NODE_SET_METHOD(exports, "hello", Method);
    }

    NODE_MODULE(addon, init)

注意：所有的 Node 插件必须输出一个初始化函数:

    void Initialize (Handle<Object> exports);
    NODE_MODULE(module_name, Initialize)

`NODE_MODULE` 之后的代码没有分号，因为它不是一个函数 (参见`node.h`)。

`module_name` 必须和二进制文件名字一致 (后缀是 .node)。

源文件会编译成 `addon.node` 二进制插件。 为此我们创建了一个很像 JSON 的 `binding.gyp` 文件， 它包含配置信息，这个文件用[node-gyp](https://github.com/TooTallNate/node-gyp)编译。

    {
      "targets": [
        {
          "target_name": "addon",
          "sources": [ "hello.cc" ]
        }
      ]
    }

下一步创建一个 `node-gyp configure` 工程，在平台上生成这些文件。

创建后，在`build/`文件夹里拥有一个 `Makefile` (Unix 系统) 文件或者 `vcxproj` 文件
(Windows 系统)。 接着调用 `node-gyp build` 命令编译，生成 `.node` 文件。 这些文件在 `build/Release/` 目录里。

现在，你能在 Node 工程中使用这些 2 进制扩展插件，在 `hello.js` 中声明`require`之前编译的`hello.node`:

    // hello.js
    var addon = require('./build/Release/addon');

    console.log(addon.hello()); // 'world'

更多的信息请参考<https://github.com/arturadib/node-qt> 。


## 插件模式

下面是一些 addon 插件的模式，帮助你开始编码。[v8 reference](http://izs.me/v8-docs/main.html) 文档里包含 v8 的各种接口，[Embedder's Guide](http://code.google.com/apis/v8/embed.html)这个文档包含各种说明，比如 handles, scopes, function templates, 等等。

在使用这些例子前，你需要先用 `node-gyp` 编译。
创建`binding.gyp` 文件:

    {
      "targets": [
        {
          "target_name": "addon",
          "sources": [ "addon.cc" ]
        }
      ]
    }

将文件名加入到 `sources` 数组里就可以使用多个 `.cc` 文件，例如 :

    "sources": ["addon.cc", "myexample.cc"]

准备好 `binding.gyp` 文件后, 你就能配置并编译插件:

    $ node-gyp configure build


### 函数参数

从以下模式中解释了如何从 JavaScript 函数中读取参数，并返回结果。仅需要一个`addon.cc`文件:

    // addon.cc
    #include <node.h>

    using namespace v8;

    void Add(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.Length() < 2) {
        isolate->ThrowException(Exception::TypeError(
            String::NewFromUtf8(isolate, "Wrong number of arguments")));
        return;
      }

      if (!args[0]->IsNumber() || !args[1]->IsNumber()) {
        isolate->ThrowException(Exception::TypeError(
            String::NewFromUtf8(isolate, "Wrong arguments")));
        return;
      }

      double value = args[0]->NumberValue() + args[1]->NumberValue();
      Local<Number> num = Number::New(isolate, value);

      args.GetReturnValue().Set(num);
    }

    void Init(Handle<Object> exports) {
      NODE_SET_METHOD(exports, "add", Add);
    }

    NODE_MODULE(addon, Init)

可以用以下的 JavaScript 代码片段测试：

    // test.js
    var addon = require('./build/Release/addon');

    console.log( 'This should be eight:', addon.add(3,5) );


### 回调Callbacks

你也能传 JavaScript 函数给 C++ 函数，并执行它。 在 `addon.cc` 中:

    // addon.cc
    #include <node.h>

    using namespace v8;

    void RunCallback(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<Function> cb = Local<Function>::Cast(args[0]);
      const unsigned argc = 1;
      Local<Value> argv[argc] = { String::NewFromUtf8(isolate, "hello world") };
      cb->Call(isolate->GetCurrentContext()->Global(), argc, argv);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", RunCallback);
    }

    NODE_MODULE(addon, Init)

注意，这个例子中使用了 `Init()` 里的 2 个参数，`module`对象是第二个参数。它允许 addon 使用一个函数完全重写 `exports`。

可以用以下的代码来测试：

    // test.js
    var addon = require('./build/Release/addon');

    addon(function(msg){
      console.log(msg); // 'hello world'
    });


### 对象工厂

在 `addon.cc` 模式里，你能用 C++ 函数创建并返回一个新的对象，这个对象所包含的 `msg` 属性是由`createObject()` 函数传入:

    // addon.cc
    #include <node.h>

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<Object> obj = Object::New(isolate);
      obj->Set(String::NewFromUtf8(isolate, "msg"), args[0]->ToString());

      args.GetReturnValue().Set(obj);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", CreateObject);
    }

    NODE_MODULE(addon, Init)

使用 JavaScript 测试:

    // test.js
    var addon = require('./build/Release/addon');

    var obj1 = addon('hello');
    var obj2 = addon('world');
    console.log(obj1.msg+' '+obj2.msg); // 'hello world'


### 工厂模式

这个模式里展示了如何创建并返回一个 JavaScript 函数，它是由 C++ 函数包装的 :

    // addon.cc
    #include <node.h>

    using namespace v8;

    void MyFunction(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      args.GetReturnValue().Set(String::NewFromUtf8(isolate, "hello world"));
    }

    void CreateFunction(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, MyFunction);
      Local<Function> fn = tpl->GetFunction();

      // omit this to make it anonymous
      fn->SetName(String::NewFromUtf8(isolate, "theFunction"));

      args.GetReturnValue().Set(fn);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      NODE_SET_METHOD(module, "exports", CreateFunction);
    }

    NODE_MODULE(addon, Init)

测试:

    // test.js
    var addon = require('./build/Release/addon');

    var fn = addon();
    console.log(fn()); // 'hello world'


### 包装 C++ 对象

以下会创建一个 C++对象的包装`MyObject`，这样 他就能再 JavaScript 中用 `new` 实例化。首先在`addon.cc`中准备主要模块:

    // addon.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    void InitAll(Handle<Object> exports) {
      MyObject::Init(exports);
    }

    NODE_MODULE(addon, InitAll)

接着在 `myobject.h` 创建包装，它继承自 `node::ObjectWrap`:

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init(v8::Handle<v8::Object> exports);

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

在 `myobject.cc` 中实现各种暴露的方法，通过给构造函数添加 prototype 属性来暴露 `plusOne` 方法:

    // myobject.cc
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init(Handle<Object> exports) {
      Isolate* isolate = Isolate::GetCurrent();

      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      // Prototype
      NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

      constructor.Reset(isolate, tpl->GetFunction());
      exports->Set(String::NewFromUtf8(isolate, "MyObject"),
                   tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
      obj->value_ += 1;

      args.GetReturnValue().Set(Number::New(isolate, obj->value_));
    }

测试:

    // test.js
    var addon = require('./build/Release/addon');

    var obj = new addon.MyObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13

### 包装对象工厂

当你想创建本地对象，又不想在 JavaScript 中严格的使用 `new` 初始化的时候，以下方法非常实用。

    var obj = addon.createObject();
    // instead of:
    // var obj = new addon.Object();

在 `addon.cc` 中注册 `createObject` 方法:

    // addon.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      MyObject::NewInstance(args);
    }

    void InitAll(Handle<Object> exports, Handle<Object> module) {
      MyObject::Init();

      NODE_SET_METHOD(module, "exports", CreateObject);
    }

    NODE_MODULE(addon, InitAll)

在 `myobject.h` 中有静态方法 `NewInstance`，他能实例化对象 （它就像 JavaScript 的 `new`):

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

这个实现方法和 `myobject.cc` 类似:

    // myobject.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init() {
      Isolate* isolate = Isolate::GetCurrent();
      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      // Prototype
      NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

      constructor.Reset(isolate, tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> instance = cons->NewInstance(argc, argv);

      args.GetReturnValue().Set(instance);
    }

    void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
      obj->value_ += 1;

      args.GetReturnValue().Set(Number::New(isolate, obj->value_));
    }

测试:

    // test.js
    var createObject = require('./build/Release/addon');

    var obj = createObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13

    var obj2 = createObject(20);
    console.log( obj2.plusOne() ); // 21
    console.log( obj2.plusOne() ); // 22
    console.log( obj2.plusOne() ); // 23


### 传递包装对象

除了包装并返回 C++ 对象，你可以使用 Node 的 `node::ObjectWrap::Unwrap` 帮助函数来解包。在下面的 `addon.cc` 中，我们介绍了一个 `add()` 函数，它能获取2个 `MyObject`对象：

    // addon.cc
    #include <node.h>
    #include <node_object_wrap.h>
    #include "myobject.h"

    using namespace v8;

    void CreateObject(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);
      MyObject::NewInstance(args);
    }

    void Add(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
          args[0]->ToObject());
      MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
          args[1]->ToObject());

      double sum = obj1->value() + obj2->value();
      args.GetReturnValue().Set(Number::New(isolate, sum));
    }

    void InitAll(Handle<Object> exports) {
      MyObject::Init();

      NODE_SET_METHOD(exports, "createObject", CreateObject);
      NODE_SET_METHOD(exports, "add", Add);
    }

    NODE_MODULE(addon, InitAll)

介绍 `myobject.h` 里的一个公开方法，它能在解包后使用私有变量:

    // myobject.h
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>
    #include <node_object_wrap.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
      inline double value() const { return value_; }

     private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
      static v8::Persistent<v8::Function> constructor;
      double value_;
    };

    #endif

`myobject.cc` 的实现方法和之前的类似:

    // myobject.cc
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Persistent<Function> MyObject::constructor;

    MyObject::MyObject(double value) : value_(value) {
    }

    MyObject::~MyObject() {
    }

    void MyObject::Init() {
      Isolate* isolate = Isolate::GetCurrent();

      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
      tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      constructor.Reset(isolate, tpl->GetFunction());
    }

    void MyObject::New(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      if (args.IsConstructCall()) {
        // Invoked as constructor: `new MyObject(...)`
        double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
        MyObject* obj = new MyObject(value);
        obj->Wrap(args.This());
        args.GetReturnValue().Set(args.This());
      } else {
        // Invoked as plain function `MyObject(...)`, turn into construct call.
        const int argc = 1;
        Local<Value> argv[argc] = { args[0] };
        Local<Function> cons = Local<Function>::New(isolate, constructor);
        args.GetReturnValue().Set(cons->NewInstance(argc, argv));
      }
    }

    void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
      Isolate* isolate = Isolate::GetCurrent();
      HandleScope scope(isolate);

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> instance = cons->NewInstance(argc, argv);

      args.GetReturnValue().Set(instance);
    }

测试:

    // test.js
    var addon = require('./build/Release/addon');

    var obj1 = addon.createObject(10);
    var obj2 = addon.createObject(20);
    var result = addon.add(obj1, obj2);

    console.log(result); // 30
