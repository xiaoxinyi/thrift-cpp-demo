# thrift cpp demo

## 运行 demo 的步骤

1.克隆仓库

```sh
git clone https://github.com/xiaoxinyi/thrift-cpp-demo thrift-cpp
```

2.利用`tutorial.thrift`和`shared.thrift`生成需要的接口文件。注意，这两个文件都需要使用，在`tutoial.thrift`中引用了(继承了)`shared.thrift`。

```sh
thrift -r --gen cpp tutorial.thrift
```

3.利用`CMakeLists.txt`生成demo的可执行文件。

```sh
mkdir -p build
cd build
cmake ..
make
```

4.运行服务器端和客户端的的程序。

```sh
./server &
./client
```

## CMakeLists.txt说明

主要生成3个目标文件，分别是`libtutorialgencpp.a`，`server`和`client`。

在链接生成这三个目标文件的时候需要链接`thrift`库和`pthread`库。我的`thrift`库的路径是`/usr/local/lib/libthrift.a`。所以必须加上以下的两句：

```cmake
# 比如链接生成目标tutorialgencpp
target_link_libraries(tutorialgencpp /usr/local/lib/libthrift.a)
target_link_libraries(tutorialgencpp pthread)
```

NOTE:因为需要用到`boost`库，添加`include_directories("/usr/include/boost/")`。

## thrift说明

定义在`tutorial.thrift`中的服务接口：

```
service Calculator extends shared.SharedService {

  /**
   * A method definition looks like C code. It has a return type, arguments,
   * and optionally a list of exceptions that it may throw. Note that argument
   * lists and exception lists are specified using the exact same syntax as
   * field lists in struct or exception definitions.
   */

   void ping(),

   i32 add(1:i32 num1, 2:i32 num2),

   i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),

   /**
    * This method has a oneway modifier. That means the client only makes
    * a request and does not listen for any response at all. Oneway methods
    * must be void.
    */
   oneway void zip()

}
```

从这个文件中，我们可以看到`Caculator`服务继承了`shared.SharedService`服务。

我们可以从`shared.thrift`中看到，它声明了`SharedService`服务的所有接口。

```
service SharedService {
  SharedStruct getStruct(1: i32 key)
}
```

现在我们来看一下总共这四个接口：其中3个在`Caculator`服务中声明，还有一个在`SharedService`服务中声明。其中`Caculator`服务所对应的类是`gen-cpp/Caculator.h`中定义的叫`CalculatorIf`的抽象类。同理`SharedService`服务所对应的类是`gen/SharedService.h`中定义的叫`SharedServiceIf`的抽象类。根据两个`.thrift`文件，我们可以推断出，这两个抽象类存在继承关系，`CaculatorIf`类继承了`SharedServiceIf`类。

我们分别来看这两个抽象类：

`CaculatorIf`类

```c++
 class CalculatorIf : virtual public  ::shared::SharedServiceIf {
  public:
   virtual ~CalculatorIf() {}

   /**
    * A method definition looks like C code. It has a return type, arguments,
    * and optionally a list of exceptions that it may throw. Note that argument
    * lists and exception lists are specified using the exact same syntax as
    * field lists in struct or exception definitions.
    */
   virtual void ping() = 0;
   virtual int32_t add(const int32_t num1, const int32_t num2) = 0;
   virtual int32_t calculate(const int32_t logid, const Work& w) = 0;

   /**
    * This method has a oneway modifier. That means the client only makes
    * a request and does not listen for any response at all. Oneway methods
    * must be void.
    */
   virtual void zip() = 0;
 };
```

`SharedServiceIf`类

```c++
class SharedServiceIf {
 public:
  virtual ~SharedServiceIf() {}
  virtual void getStruct(SharedStruct& _return, const int32_t key) = 0;
};
```

从这两个抽象类定义中，我们看到类`CaculatorIf`虚继承于`SharedServiceIf`。以及声明了4个纯虚函数，与两个`.thrift`文件中的接口一一对应。

### thrift服务器端

服务器端我们需要写一个自己的类继承`CaculatorIf`，在这个demo中写了一个`CaculatorHandler`这样一个类，这个类中实现了祖先类中所有的虚函数，即4个接口函数。

在demo中，还实现了一个叫`CalculatorCloneFactory`的类，这个类虚继承于`CalculatorIfFactory`。是否实现这个类，取决于你要使用的服务类别。具体可以参考`CppServer.cpp`文件。



