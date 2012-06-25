---
layout:       post
title:        "Ruby中的方法、过程、Lambda和闭包(学习笔记)"
description: 
banner: 
categories: 
- ruby learning notes
---

#Method
>方法是一个有名代码块，与一个或多个对象相关联的参数化代码。调用方法时需要给出方法名、所在对象以及零个或多个与有名参数对应的参数值。方法中的最后一个表达式作为方法的返回值。

##取消方法的定义
取消方法的定义可以使用undef，但是如果想要取消单例方法，不能使用它。在方法或模块中可以使用undef_method。
    
    def  hello 
      "hello world"
    end

    undef hello

##定义方法的习惯

方法的名词一般都是以小写字母命名的，如果方法是多个名词，那么用下划线来分割。方法的定义有一个需要注意的地方：
+ 如果方法名以`?`结尾，那么此类的方法称为"断言式"方法，通常情况下返回false,true或nil，但也不尽然。比如Numberic的nonzero?方法，如果为0则返回nil，否则返回原来的值。当然Ruby中以false和nil判断为False,除此之外的值都是认为是True的。
+ 方法名以`!`结尾的方法，则提示该方法是比较危险的，会修改对象的实例变量的值。以`!`定义的方法通常是可变方法。也有一些方法不以`!`结尾，但是属于可变方法。

##操作符方法

Ruby本身是很对象化的编程语言，所以操作符其实也是一种方法。对于int,float这些类的操作符是不可继承的。

如何定义这些方法呢?

    def +(other)
      self.age + other
    end

    def [](index)
      self.ary[index]
    end

##方法的重命名

人有七名八姓，Ruby有方法重命名。方法重命名可以使用不同的方法名对某个方法进行调用。亦然以我们更加容易理解的名称来命名某些方法。比如Range类的include?和member？，Kernel类的raise和fail。
方法重命名并不是进行方法的重载，仅仅是对方法名的重新命名。

    def  hello
      "say hello"
    end

    alias say_hello hello

    def hello
      "can u say hello?"
    end

##我们常见的括号呢？

先举个例子：
    
    puts "hello world"
    puts ("hello world")

上面的方法调用有区别吗？在形式上是有区别的，但是实质上是一样的。因为在Ruby中通常是忽略括号的，如果一个方法只有一个甚至没有参数时，括号可以省略；如果方法有多个参数时，原括号也是可以省略的，但是不能说很常见。

但是什么时候必须要括号呢？在有可能引起歧意的时候。
比如"f g x y"，通常ruby会解释成"f(g(x,y))"。为了消除歧义性或为了方便人们的阅读，必要的括号还是很重要的。

看下面的例子:
    
    seq(2+2) * 2
    seq (2+2) * 2

上面的例子有区别吗？有。第一个括号表示方法的调用，2+2是方法的参数，而后者是用于分组，seq的参数为 （2+2）* 2。也就是说如果括号紧跟着一个方法名，那么括号里面的变量都是认为是方法的参数的，如果有空格之类的，那么连同括号，都是方法的参数。

    puts(sum 2,2)
    puts (sum 2,2)

同样的例子。上面的调用是有歧义的，与`puts sum 2, 2`是没有任何区别的。所以为了正确表达，需要用括号进行分组，在方法名后面用空格分隔。

##方法的参数

1.带默认参数的方法

    def hello(name, family='Wei')
      "hello, #{name} family"
    end

    puts hello('david')
    puts hello('david','Green')

方法的默认参数不一定是常量，还可以是任意的表达式，也可以是实例变量的引用，还可以是对前面定义的参数的引用。参数的默认值是在运行时才进行求值，而不是在解析时。

在Ruby 1.9中放开了原来1.8中规定的默认参数必须在参数列表在最后面的限制，但是仍然要求默认参数必须连续，不能中间插入其它的变量。

2.任意个数的参数

任意个数的参数需要在参数名称前面加上*,它表示这部分传递的是一个数组。btw.数组的个数也有可能为0。

用`*`打头的参数不能超过一个,一般该参数是放在其它参数的后面，在1.9中稍微放开了些，可以放在其它普通参数的前面，但是必须放在默认参数的后面和含`&`打头参数的前面。

如果传递的参数为Hash，且Hash为最后一个参数(可以有&代码块)，那么Hash的大括号可以去掉。如果在传递Hash参数的时候是省略了方法的括号的，那么Hash的大括号也要去掉，因为Ruby会认为你给它传递的不是Hash，而是一个代码块，从而运行报错。

3. 代码块参数

    def seq(n,m,o)
      yield n * m + 0
    end

    seq(2,3,4){ |x| puts x } 

上面就是一个比较简单的调用代码块参数的例子。只不过代码块参数不是明确写在方法的参数列表中，而是通过yield来进行调用代码块参数。代码块有一个特点:`匿名性`。它们没有名字，只是通过一个关键字对它进行调用。如果需要明确地方式来控制一个代码块，可以在方法的最后面加上一个参数，并使用`&`作参数的前缀。这个参数的值有了些许变化，它的值(含&)是Proc对象，在调用的时候不是简单的使用yield来调用，而是需要用Proc的call方法来调用。

    def seq(n,m,o,&b)
      b.call (n * m + o)
    end

    seq(2,3,4){ |x| puts x }

如果上面例子中的b本身是一个Proc对象，那么`&`是不要的。其它的调用方法也是一样的。

    def seq(n,m,o,b)
      b.call (n * m + o)
    end

    proc = Proc.new { |x| puts x }

    seq(2,3,4,proc)

上面说到了，可以使用`&`将一个代码块转换为proc对象，那么proc对象是否可以变为代码块呢？档案是可以。

    a,b = [1,2,3],[4,5] #并行赋值表达式

    sum = a.inject(0) { |total, x| total + x }
    sum = b.inject(sum) {|total, x| total + x}

上面的代码块是相同的，在运行时需要解析2次，所以我们可以将之定义为一个一个Proc对象，调用它2次。

    a,b = [1,2,3],[4,5]

    p = Proc.new { |total, x| total + x }

    sum = a.inject(0, &p)
    sum = b.inject(sum, &p)

`&`通常出现在一个Proc对象之前，它能支持所有支持to_proc方法的对象，像Method,Symbol都有这个方法，所以我们可以这样使用：

    words = ['i','love','ruby']
    uppercase = words.map &:upcase
    uppercase = words.map {|c| c.upcase} #this line is the same as the one above.

##Procs and Lambdas

代码块是Ruby的一种句法结构，而不是对象，也不能像对象一样操作。不过，我们可以创建对象来表示代码块。创建对象的方式有proc和lambda，前者的行为与代码块类似，后者的行为则与方法类似。

前面我们举的例子里面就包含着创建Proc对象的方法，而且后面都是包含一个代码块的。实际上，如果不包含代码块，那么它默认指向所在方法所关联的代码块；如果在创建的时候包含着代码块，那么它指向它所关联的代码块。

另外一种创建Proc对象的方法是使用lambda方法，它是Kernel模块的方法，返回的Proc对象是一个lambda而非proc。lambda方法不带参数，但是在调用时必须关联一个代码块。

    is_positive = lambda {|x| x > 0}

第三种是Kernel模块的proc方法，它在1.8中和lambda是同义的，但是在1.9中它是Proc.new的同义词。    
>在Ruby 1.9中，Ruby支持了一种全新的方法，将lambda作为字面量。在1.8中定义lambda是这样的：

    is_positive = lambda {|x| x > 0}

>而在Ruby 1.9 中我们我们可以这样做：
+ 把lambda方法名替换为 `->`
+ 把大括号内的变量，移到{}的前面。
+ 把参数列表的变量分隔符从`||`替换为`()`

    is_positive = -> (x){ x > 0 }

在新的定义语法中得到的好处是：我们可以定义默认参数了:`zoom = ->(x,y,factor=2){x*factor,y*factor}`。另外它还很简洁，产生的代码块，如果要传递给一个需要Proc对象的方法时，需要添加`&`。

    data.sort {|a,b| a - b}
    #变化后:
    data.sort &->(a,b){ a - b }

最小的lambda代码为`->{}`，它返回的是nil。

**调用proc和lambda的方式除了使用call，还可以使用`[]`和`.()`**
Proc类定义了数组访问操作符，它的工作方式和call是相同的，所以可以使用`[]`来调用proc对象。还可以使用后面的`.()`的方式来调用。
    is_positive[3]
    is_positive.(3)

如何知道一个Proc对象所需的参数个数呢？使用arity(来源于一元(unary),二元(binary),三元(ternary))

如果参数中不包含`*`开头的参数，那么arity返回的是不小于0的参数个数。

如果参数中包含`*`的参数，那么arity返回的是-n-1的负数，负数说明该方法可以接受额外的参数。-n-1是n的互补数。那么有多少个参数是必须需要的呢？取反-1.即-(-n - 1) - 1=n

##Lambda和Proc的区别

proc是代码块的对象形式，它的行为就像一个代码块；而lambda的行为略微不同，它的行为更像方法而非代码块。调用一个proc则像对代码块进行yield，而调用lambda则像调用了对象的方法。在Ruby 1.9中可以通过Proc的实例方法`lambda?`来判断某个实例是一个proc还是lambda。如果返回为真，则为lambda，否则为proc。

除了上面的一些区别，还有什么地方不一样呢？

1 代码块、proc和lambda中的return语句

在一个代码块中的return语句不仅会使代码块返回,还会导致调用它的方法返回。

    def test
      puts "begin of the block"
      10.times { |x| return if x == 3 }
      puts "end of the block"
    end

    test #the result will not output the 'end of the block'

proc与代码块类似，因此在proc中执行return，它也会尝试从代码块所在的方法中返回。

    def test
      puts "begin of the block"
      p = Proc.new {|x| return if x == 3}
      p.call
      puts "end of the block"
    end

    test #it is the same result as the one above.

proc经常在不同方法之间传递，而如果proc中使用return语句可能会达不到预期的目的，因为在调用proc时，在句法上包含该proc的方法已经返回。

    def  procBuilder(msg)
      Proc.new { puts "#{msg}"; return }
    end

    def test
      puts "begin of the block"
      p = procBuilder('hello')
      p.call  #LocalJumpError: unexpected return
      puts "end of the block"
    end

    test

除了去掉return语句，还可以使用lambda来修复这个问题。前面说到了lambda在调用时更像是一个方法，所以return语句只会从lambda中返回，而调用它的方法不会。

    def procBuilder(msg)
      ->() { puts "#{msg}"; return }
    end

    def test
      puts "begin of the block"
      p = procBuilder('msg')
      p.call
      puts "end of the block"
    end

    test

顶级的break、redo和next也有相同的行为。在proc和lambda中，retry是禁止使用的:因为它会导致死循环的LocalJumpError异常。

另外在参数上，proc会自动处理传递的参数，少于规定个数的，会自动赋值为nil; 多余规定的参数会被丢弃掉；数组会自动拆开；多个也可以进行打包。 而lambda在处理上则没有如此大的灵活性，参数个数要严格遵守。

#Closures闭包

在Ruby中，proc和lambda都是闭包。那么神马是闭包呢? 闭包表示一个对象即是一个可以调用的函数，也是绑定在这个函数上的变量。(好晦涩Sigh).当创建一个proc或lambda时，得到的Proc对象不仅包含了可执行的代码块，还绑定了代码块中所使用的所有变量。

闭包的要素包括:对象、函数、绑定的变量。 首先它必须是一个对象，第二它还需要可以被调用，最后它的所有变量都是已经被绑定的。

    def multiplier(n)
      ->(data) { data.collect { |x| x * n } }
    end

    p = multiplier(2)
    p.call([1,2,3,4]) #[2, 4, 6, 8]

multiplier方法返回一个lambda，它可以在方法返回后继续访问n,它被称为一个闭包。

闭包并不持有它引用的参数，但是持有它所使用的变量并延长了其生命周期。lambda或proc在创建时并不静态绑定所使用的变量，而是在运行时才回去查找变量的值。
    
    def test(initValue=nil)
      value = initValue
      getter = -> { value }
      setter = ->(x) { value = x }
      return getter,setter
    end

    getX, setX = test(2)
    puts getX[] #the same as getX.call
    setX[3]
    puts getX[]

下面让我们看一个比较晦涩些的代码:
  
    def test(*args)
      x = nil
      args.map { |x| lambda {|y| x * y} }
    end

    a, b=test(2,3)
    puts a.call(3) #what is the rest? yes, it is 6.

上面的代码返回的是一组lambda，所以a,b对应的lambda中的x分别为2和3。所以a在调用的时候就为2*3=6了。

Proc定义了一个binding方法，表示该闭包所使用的绑定，返回一个Binding对象。它作为eval的第二个参数，能为eval函数执行时的代码提供上下文环境。使用Binding对象和eval可以让我们获得闭包的后门，可以修改已经绑定的某些值。

    def multiplier(n)
      ->(data) { data.collect{ |x| x * n } }
    end

    double = multiplier(2)
    puts double([1,2,3])

    eval("n=3", double.binding)
    puts double([1,2,3]) #it will be 3,6,9

#Methods

Ruby的方法和代码块都是可执行的语言构件，但它们不是对象。proc和lambda是代码的对象版本，它们可以被执行。Ruby的方法也可以使用Method的实例来表示，但是通常比直接调用效率要低。Method对象比proc和lambda使用频繁度要低。

Method类不是Proc类的子类，但是它们在行为上很相似.Method对象使用`call`来表示调用，也有`arity`方法。Method对象的call方法在调用时使用的是**invocation**,这一点和lambda相同，而proc使用的是yield调用。

Method对象与Proc对象工作方式非常类似，因此常用来替代Proc对象。在需要Proc对象的地方，可以使用`Method.to_proc`把Method对象转换为Proc对象。所以我们可以在Method对象前面加`&`，然后使用这个对象取代代码块作参数传递了。

    def sequre(x)
      x ** 2
    end

    puts (1..10).map(&method(:sequre))

>Method和Proc的一个重要区别是：Method对象不是闭包。

Ruby的方法不会访问在其范围之外的局部变量，唯一绑定的值是`self`。在Ruby 1.9中添加了3个新方法:name、owner、receiver。name返回方法的名字。owner返回定义该方法的类。receiver返回该方法绑定的对象。

还有一种无绑定的方法。此类方法的获取方式有2种。一种是通过instance_method方法获取，一种是通过unbind获取。

    myplus = Fixnum.instance_method("+")
    myplus.bind(3) #绑定对象
    result = myplus.call(4) #7
    
    other_plus = myplus.unbind.bind(3)
    result = other_plus.call(5) #8


