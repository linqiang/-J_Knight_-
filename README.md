# 答J_Knight_ 出的面试题（基础题）
### 分类和扩展有什么区别，可以分别用来做什么？分类有哪些局限性？分类的结构体中有哪些成员
#### 分类(category)和扩展(Extension)的区别和作用
1. 类别只能扩充方法，不能扩展属性和成员变量（如果包含成员变量会直接报错）；如果分类中声明了一个属性，那么分类只会生成这个属性的set、get方法声明，也就是不会有实现,类别用来扩展当前类，为类添加方法扩展
2. 扩展有时也成为匿名类别，能为某个类附加额外的属性，成员变量，方法声明，一般的将类扩展直接写在.m文件中，而不单独建立类扩展文件，一般的私有属性和方法写到类扩展，和类别相似，但是小括号里面没有扩展的名字，就像匿名的类别

#### 分类的局限性
1. 无法向现有的类中添加新的实例变量(类别没有位置容纳实例变量)。Category一般情况下只能为类提供方法的扩展，不提供属性的扩展。但是利用RunTime也可以在Category中添加属性。
2. 方法名称冲突，即类别中的新方法的名称与现有类中方法名称重名，类别具有更高的优先级，类别中的方法将完全取代现有类中的方法(再也无法访问现有类中的同名方法)。

#### 分类的结构体中有哪些成员
1. 这里暂时不太清楚

### 讲一下atomic的实现机制；为什么不能保证绝对的线程安全（最好可以结合场景来说）？
1. `atomic`给属性的 `get/set`方法加锁，来实现读写方法的互斥。并不是给线程加锁所以对于线程来讲并不是安全的。
2. 比如 `@property(atomic,strong)NSMutableArray *arr`;如果一个线程循环读数据，一个线程循环写数据，肯定会产生内存问题。因为它和 `setter,getter` 没有关系。

### 被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？
#### 被weak修饰的对象在被释放时发生了什么
1. runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。
2. sideTable结构图[iOS 底层解析weak的实现原理（包含weak对象的初始化，引用，释放的分析）](http://www.cocoachina.com/ios/20170328/18962.html)
![](media/15324945556706/15324989476887.png)



### 关联对象有什么应用，系统如何管理关联对象？其被释放的时候需要手动将所有的关联对象的指针置空么？
#### 关联对象的作用
1. 使OC的某个对象通过唯一的key连接到一个类的实例上
2. 被释放时不需要将所有关联对象指针置空

#### 系统如何管理关联对象
[关联对象](https://www.jianshu.com/p/c68cc81ef763)

### KVO的底层实现？如何取消系统默认的KVO并手动触发（给KVO的触发设定条件：改变的值符合某个条件时再触发KVO）？
#### KVO底层实现
1. 在调用KVO的时候，实际上是自动生成了一个   NSKVONotify_类名，重写set方法，在runtime中实现

#### 手动实现KVO
1. [手动实现KVO](http://tech.glowing.com/cn/implement-kvo/)

### Autoreleasepool所使用的数据结构是什么？AutoreleasePoolPage结构体了解么？
1. 暂时不会需要深入调研一下

### class_ro_t 和 class_rw_t 的区别？
* [深入解析 Objective-C 中方法的结构](https://blog.csdn.net/fishmai/article/details/71157861)
* 类的方法、属性以及协议在编译期间存放到了“错误”的位置，直到 realizeClass 执行之后，才放到了 class_rw_t 指向的只读区域 class_ro_t，这样我们即可以在运行时为 class_rw_t 添加方法，也不会影响类的只读结构。

* 在 class_ro_t 中的属性在运行期间就不能改变了，再添加方法时，会修改 class_rw_t 中的 methods 列表，而不是 class_ro_t 中的 baseMethods

### iOS 中内省的几个方法？class方法和objc_getClass方法有什么区别?
#### 常用的内省方法
1. `isKindOfClass:Class` 检查对象是否是那个类或者其继承类实例化的对象
2. `isMemberOfClass:Class` 检查对象是否是那个类但不包括继承类而实例化的对象
3. `respondToSelector:selector` 检查对象是否包含这个方法
4. `conformsToProtocol:protocol` 检查对象是否符合协议，是否实现了协议中所有的必选方法。

#### class 和 objc_getClass方法的区别
* object_getClass(obj)返回的是obj中的isa指针；
* [obj class]则分两种情况：
    1. 是当obj为实例对象时，[obj class]中class是实例方法：- (Class)class，返回的obj对象中的isa指针；
    2. 当obj为类对象（包括元类和根类以及根元类）时，调用的是类方法：+ (Class)class，返回的结果为其本身。


###  为什么在block外部使用__weak修饰的同时需要在内部使用__strong修饰？
 * 在block中调用self会引起循环引用，但是在block中需要对weakSelf进行strong,保证代码在执行到block中，self不会被释放，当block执行完后，会自动释放该strongSelf；
    
### RunLoop的作用是什么？它的内部工作机制了解么？（最好结合线程和内存管理来说）
* [深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/#base)

### 哪些场景可以触发离屏渲染？
* shouldRasterize（光栅化）
* masks（遮罩）
* shadows（阴影）
* edge antialiasing（抗锯齿）
* group opacity（不透明）
* 复杂形状设置圆角等
* 渐变
