C++主要是参考**《深度探索C++对象模型》**这本书来复习的，这本书把C++对象模型讲解的非常透彻，之前也阅读过了**《C++ Primer》**和**《Effective C++》**，后面两本书也讲的很好。 下面主要是**《深度探索C++对象模型》**中的笔记。 

**member functions**虽然在class的声明之内，但却不会出现在object之中，每一个**non-inline member function** 只会诞生一个函数实例。至于每一个“拥有零个或一个定义”的**inline function** 则会在其每一个使用者身上产生一个函数实例。

C++在布局以及存取时间上的主要额外负担是由**virtual**引起的，包括：  
1. **virtual function机制** ，用于支持一个有效率的**“执行期绑定(runtime binding)”**。  
2. **virtual base class** ，用以实现**“多次出现在继承体系中的base class，有一个单一而被共享的实例”** 。


> C++有两种**class data members**：static和非static。  
> 三种**class member functions**:static,non-static,virtual。


对于object的**多态**操作，要求此object必须经由一个**pointer** 或者**reference**来存取。

需要多少内存才能够表现一个class object，一般而言主要有：  
1. 其**non-static data members**的总和大小  
2. 加上任由由于**alignment** 的需求而填补上去的空间，可能存于members之间，也可能存于集合体边界。  
3. 加上为了支持**virtual** 而由内部产生的任何额外负担


有4种情况，会造成“编译器”必须为未声明construct的class合成一个default construct:  
1. 类里面存在member object时  
2. 类存在继承体系，base class的default construct  
3. 类里面存在**virtual function** 时  
4. 类继承体系中出现**virtual base class**时  

有3种情况会以一个object的内容作为另一个object的初值：  
1. 以一个object显式地初始化操作  
2. 当object当做参数传递给某一个函数  
3. 函数返回object  

> 一个class object可用两种方式复制得到，一种是**被初始化（copy construct）** ,另外一种是**被指定（assignment）** 。

什么时候一个class不会呈现出一种**“bitwise copy semantics”** 呢？有4种情况：  
1. 当class内含一个member objct，而后者的class声明有一个copy constructor(不论是被class设计者显式地声明，或者是被编译器合成)。  
2. 当class继承自一个**base class** ，而后者存在一个**copy constructor**   
3. 当类声明一个或多个**virtual functions** 时  
4. 当class 派生自一个继承串链，其中有一个或多个**virtual base classes** 时  

编译器间的两个**程序扩张操作（只要有一个类声明了一个或多个virtual functions就会如此）** ：   
1. 增加一个**virtual function table (vtbl)** ,内含每一个有用的**virtual function** 的地址   
2. 一个指向**virtual function table** 的指针（**vptr**），安插在每一个class object内  

当class object以“相同class的另一个object”作为初始，其内是以所谓的default memberwise initialization手法完成的，也就是把每一个内建的或者派生的data member的值，从某一个object拷贝一份到另外一个object上，不过它并不会拷贝其中的**member class object** ，而是以递归的方式施行**memberwise initialization** 。

在下列情况下，为了使程序能够被顺利编译，必须使用一个**member initialization list** ：  


1. 当初始化一个**reference member** 时  
2. 当初始化一个**const member** 时  
3. 当调用一个**基类** 的constructor,而其有一组参数时  
4. 当调用一个**member class** 的 **constructor** 时，而它拥有一组参数时  




> **member list** 中的顺序是由**member** 中的声明顺序来决定的  



>编译器会对**initialization list** 一一处理并可能重新排序，以反映出members 的声明顺序，它会安插一些代码到constructor中，并置于任何**explicit user code** 之前。   
1.   编译器自动加上的额外**data members** ，用于支持某些语言特性（主要是各种virtual特性）  
2.   因为**alignment** (边界调整)的需要  


static member不可能做到两点：  


1. 直接存取**non-static 数据**  
2. 被声明为**const**   

**static member function** 的主要特性是没有this指针，次要特性：  


1. 它不能够直接存取其class的**non-static members**  
2. 它不能够被声明为const，volatile或virtual。  
3. 它不需要经由class object才能调用  

单一继承的**虚成员函数** ：  
一个class只会有一个**virtual table**，每一个table内含其对应之class object中所有active virtual functions函数实例的地址，这些active virtual functions包括：  


1. 这一class所定义的函数实例，它会改写（overriding）一个可能存在的base class virtual function函数实例  
2. 继承自base class的函数实例，这是在**derived class** 决定不改写**virtual function** 时才会出现的情况  
3. 一个**per\_virtual\_called函数实例** ，它既可以扮演pure virtual function的空间保卫者，也可以当做执行期异常处理函数  

 当一个类派生时，会出现3种情况：  
1. 它可以继承base class所声明的virtual functions的函数实例，正确地说是，把函数实例的地址拷贝到**derived class** 的 **virtual talbe** 的相应slot中去  
2. 它可以使用自己的函数实例，这表示它自己的函数实例地址必须放在对应的slot中  
3. 它可以加入一个新的**virtual function**，这时候virtual table的尺寸会增大一个slot,而新的函数实例地址会被放进该slot中。


**constructor** 可能内含大量的隐藏码，因为编译器会扩展每一个constructor，扩充程度视class的继承体系而定，一般而言编译器所做的扩充操作大约如下：  
1. 记录在**member initialization list** 中的 **data member** 初始化操作会被放进constructor的函数本体，并以member的声明顺序为顺序  
2. 如果有一个member并没有出现在**member initialization list**  中，但它有一个default constructor，那么该**default constructor** 必须被调用  
3. 在那之前，如果类对象有**virtual table pointer（s）** ,它（们）必须被设定初值，指向适当的**virtual talbe(s)**  
4. 在那之前，所有上一层的**base class constructor** 必须被调用，以base class的声明顺序为顺序（与member initialization list中的顺序没有关系) 
> 如果**base class** 被列于**member initialization list** 中，那么任何显示指定的参数都应该被传递进去  
如果**base class** 没有被列于**member initialization list** 中，而它有**default constructor(或default memberwise copy constructor )** ,那么就调用之。     
如果**base class** 是多重继承的第二个或后继的**base class** ,那么this指针必须有所调整。  

5.在那之前，所有**virtual base class constructors** 必须调用，从左到右，从最深到最浅  
> 如果class被列于**member initialization list** 中，那么如果有任何显示指定的参数，都应该传递过去，若没有列于list中，而class有一个default constructor，亦调用之  
> 此外，class中每一个**virtual base class subject** 的偏移位置（offset）必须在执行期可被存取  
> 如果class object是最底层的class，其constructor可能被调用；某些用以支持这一行为的机制必须被放进来

vptr的处理：
> 在**base class constructor** 调用操作之后，但在程序员供应的代码或是**member initialization list** 中所列出的members初始化操作之前。

**constructor的执行算法如下**：  
1. 在**derived class constructor** 中，所有**virtual base classes** 及上层**base class** 的constructor会被调用  
2. 上述完成之后，对象的**vptr** 会被初始化，指向相关的**virtual tables**  
3. 如果有**member initialization list** 的话，将在constructor体内扩展开来，这必须在vptr被设定之后才做，以免有一个virtual member function被调用  
4. 最后，执行程序员提供的代码  

如果显示地拒绝把一个class object指定给定一个class object  
1. 将其声明为private(只能在member function及friend中调用)  
2. 不提供定义（调用时链接会失败）  


destructor的扩展顺序：  
1. destructor的函数本体被执行  
2. 如果class拥有member class objects，而后者拥有destructors，那么它们会以其声明顺序相反的顺序被调用  
3. 如果object内含一个vptr，那么需要重设vptr，指向适当之base class的virtual table  
4. 如果有任何直接的non-virtual base class拥有destructor，它们会以其声明顺序相反的顺便被调用  
5. 如果有任何virtual base class拥有destructor，而且前讨论的这个class是最尾端的class，那么它们会以其原来的构造顺序的相反顺序被调用

> conversion 转换 


三个著名的C++语言扩充性质，都会影响C++对象：  
1. template  
2. exception handling(EH)  
3. runtime type identification(RTTI)  

> class expression template将在编译时期而非执行期被评估，因而带来重大的效率提升。

**member functions** 不应该被“实例化” ，只有在**member functions** 被使用的时候，C++ Standard才要求它们被实例化，主要有两个原因:  
1. 空间和时间效率的考虑  
2. 尚未实现的机制  

> 不能将一个整数常数（除了0）指定给一个指针  


目前的编译器，面对一个template声明，在它被一组实际参数实例化之前，只能执行有限的错误检查，template中哪些与语法无关的错误，程序员可能认为十分明显，编译器却让它通过了，只有在特定实例被定义后，才会发生抱怨，这是目前实现技术上的一个大问题。  

Template之中，对于一个non-member name的决议结果，是根据这个name使用是否与“用以实例化该template的参数类型”有关而决定的，如果使用互不相关，那么就以“scope of the template declaration” 来决定name，如果其使用互有关联，那么就以“scope of the template instantiation” 来决定name。

一个编译器必须保持两个**scope contents**:  
1. scope of the template declaration :用于专注于一般的template class  
2. scope of the template instantiation:用于专注于特定的实例  


C++的**exception handing** 由3个主要的语汇组件构成：  
1. 一个throw子句，它在程序某处发出一个exception，被抛出去的exception可以是内建类型，也可以是使用者自定义类型  
2. 一个或多个catch子句，每一个catch子句都是一个exception handler，它用来表示说，这个子句准备处理某种类型的exception，并且在封闭的大括号区段中提供实际的处理程序。  
3. 一个try区段，它被围绕以一系列的叙述句，这些叙述句可能会引发catch子句起作用  


当一个**excpetion**发生时，编译系统必须完成以下事情：  
1. 检验发生throw操作的函数  
2. 决定throw操作是否发生的try区段内  
3. 若是，编译系统必须把exception type 拿来和每一个catch子句进行比较  
4. 如果比较后吻合，流程控制应该交给catch子句手中  
5. 如果thorw的发生并不在try区段中，或没有一个catch子句吻合，那么系统必须  
> （a)摧毁所有的active local objects   
>  (b)从堆栈中将目前的函数“unwind”掉  
> （c）进行到程序堆栈的下一个函数中去，然后重复上述步骤2~5

 
下面是《Effective C++》中的一些笔记，《Effective C++》主要介绍的是C++编程中的一些陷阱和避免的方法。  

C++中的四种编程范式（programming Paradigms）：   
1. procedure\_based 面向过程  
2. objectb\_based  基于对象
3. object\_oriented 面向对象编程
4. generics 泛型编程  

> 注意C++ 11中新增的Lambdal表达式算是对函数时编程的支持吧？？  


lhs为left-hand-side：左端  
rhs为right-hand-size：右端  

如果自己没有声明，编译器就会为它声明（编译器版本的）一个copy构造函数，一个copy assignment操作符合一个析构函数，此外，如果你没有声明任何构造函数，编译器也会为你声明一个default构造函数，所有这个函数都是public且inline。  

**default构造函数和析构函数** 主要是给编译器一个地方用来放置“藏身幕后”的代码，像是调用base class和non-static成员变量的构造函数和析构函数。注意，编译器产生的析构函数时个non-virtual，除非这个类的base class自身声明有virtual析构函数（这种情况下这个函数的虚属性virtualness，主要来自于base class）。  

copy构造函数和copy assignment操作符，编译器创建的版本只是单纯地将来源对象的每个non-static成员变量拷贝到目标对象（static成员变量不会被拷贝）。  

> reference不能更改指向不同的对象。  


如果打算在一个“内含reference成员”或“内含const成员”的类内支持赋值操作符，必须自定义copy assignment。这是由于更改reference或const成员都是不合法的。  

如果某个base class将copy assignment操作符声明为private，编译器将拒绝为其derived class 生成一个copy assignemnt操作符，毕竟编译器为derived class生成的copy assignment操作符想象中可以处理的base class成分，但它们无法调用derived class无法调用的成员函数。  

将成员函数声明为private而且故意不实现它们可以阻止copy assignment和copy构造。
如果用户企图拷贝对象，编译器会报异常。  
如果member函数或friend函数如此做，链接器会报异常。  

**public，protected，private继承**：

![](http://hi.csdn.net/attachment/201109/22/0_13166619691N3C.gif)

1. public继承
    从语义角度上来说，public继承是一种接口继承（可以理解为子类对象可以调用父类的接口，也就有可能实现多态了）
    从语法角度上来说，public继承后，关系见上图。很明显父类public成员在子类中仍然是public，所以子类对象可以调用父类的接口
 
2. protected继承
    从语义角度上来说，protected继承是一种实现继承
    从语法角度上来说，protected继承后，父类public和protected成员都变成子类的protected成员了，也就是说子类对象无法调用父类的接口，只能将父类的函数当作子类的内部实现，当然也就不符合“Liskov替换原则（LSP）”了。
 
3. private继承
    从语义角度上来说，private继承是一种实现继承
    从语法角度上来说，private继承后，父类public和protected成员都变成子类的private了，它比protected继承更严格。也就说这些父类的成员只能被继承一次，再继续继承，父类的成员就不可见了。private继承更不符合“Liskov替换原则（LSP）”了。  

**里氏替换(Liskov Substitution Principle,LSP)**：面向对象设计的基本原则之一，里氏替换原则是说，任何基类可以出现的地方，子类一定可以出现，LSP是继承复用的基石。只有当衍生类可以替换掉基类，软件单位的功能不受影响时，基类才能真正被复用，而衍生类也可能在基类的基础上增加新的行为。  

当你编写一个copying函数时（拷贝构造函数或赋值操作符）请确保：复制所有local成员变量；调用所有base clas内适当的copying函数。  


对象的布局方式和它们的地址计算方式随编译器的不同而不同。  

**inline**在大多数C++程序中是编译器行为，inline只是对编译器的一个申请，不是强制命令。这项申请可以随时隐喻提出，也可以明确提出，隐喻方式是将函数定义于class定义式内。  

编译器必须在编译期间知道对象的大小。  


C++名称遮掩规则所做的唯一事情就是：遮掩名称，至于名称是否应和相同或不同的类型，并不重要。  

**接口继承**和**实现继承**不同，在public继承下，derived class总是继承base class 的接口。  
pure virtual函数只具体制定接口继承。  
inpure virtual函数具体指定接口继承及缺省的实现继承。 
non-virtual 函数具体指定接口继承以及强制实现继承。  


在应用域（application domain），复合意味着has-a(有一个)，在实现域（implementation domain），复合意味着is implemented in terms of(根据某物实现出)。  

private继承纯粹是一种实现技术，意味implemented in terms of，private继承在软件“设计”层面上没有意义，其意义只及于软件实现层面。  

private继承意味is implement in terms of，它通常比复合的级别低，但是当继承类需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，那么实际是合理的。和复合不同，private继承可以造成empty base最优化，这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。  

声明template参数时，前缀关键字class和typename可互换。  
请使用关键字typename标示嵌套从属类型名称；  
但不得在base calss lists（基类列表）式或member initialization lists（成员初始列表）内以它作为基类修饰符。  



    
 
