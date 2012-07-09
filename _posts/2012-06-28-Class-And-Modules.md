---
layout:       post
title:        "Ruby Class And Module"
description: 
banner: 
categories: 
- ruby learning notes
---

#Class
一个类的实例可以使用`类名.new`来初始化，`new`方法会自动调用该类的`initialize`方法，但是由于`initialize`方法是类的私有方法，所以不能显式的调用它。

    class Point 
      def  initialize(x, y)
        @x = x
        @y = y
      end
    end

    p = Point.new(2, 4)

上面的实例`p`并不能直接访问里面的实例变量@x,@y，因为Ruby是面向对象的语言，所以访问这些实例变量实际上是访问与实例变量的方法而已。如果直接使用`p.x`，Ruby会告诉你:"NoMethodError: undefined method \`x\` for #<Point:0x007fc7c40b1ee8 @x=2, @y=4>"。为了能够访问里面的实例变量，我们可以定义对应的访问方法。
  
     class Point 
      def  initialize(x, y)
        @x = x
        @y = y
      end

      def x
        @x
      end

      def x=(x)
        @x = x
      end

    end

    p = Point.new(2, 4)

上面我们定义了Point对实例变量x的`getter`和`setter`方法，现在我们就可以正常访问x了。如果调用`p.x = 4`,那么这里实际调用的是`p.x=(4)`，即调用的为`x=`方法。

在Java语言中，getter和setter方法在Bean中是如此的常见，每次写都挺麻烦。那么在Ruby中有没有更好简略方法呢？有。在Ruby中有访问控制器，`attr_reader`方法定义了那些实例变量可以被外部所使用，`attr_accessor`方法定义了哪些实例变量可以被外部获取也定义了`setter`相似的功能，即它是`getter`和`setter`方法的合集。

__attr_reader和attr_accessor方法后面可以直接跟变量的名称，也可以使用Symbol，也可以使用字符串__

Ruby使用了常见的数学符号来当做方法，比如；`+`,`-`,`*`，注意，它们不是简单的符号，在Ruby中，它们其实是方法。对于减号操作`-`，有一元和二元两种形式，一元减的定义要使用`-@`，在调用的时候为`-对象.变量`。
    
    class Point 
      def  initialize(x, y)
        @x = x
        @y = y
      end

      def +(other)
        return Point.new(@x+other.x, @y+other.y)
      end

      def -@
       return Point.new(-@x,-@y)
      end

      def -(other)
        return Point.new(@x-other.x, @y-other.y)
      end

    end

**Duck Typing**：它的含义是：如果它走路像一只鸭子并且也像鸭子一样嘎嘎叫，那它就是一只鸭子。

对于这句话怎么理解呢？以我们上面的代码为例，在定义`+`方法时，我们并没有对other作类型的校验，只要other有x,y这两个方法且返回一个数值就可以。如果把Point类当做一只鸭子，而other又像Point一样拥有x,y方法，我们不管它是否真的是Point对象，那么other就是一个"鸭子"。那么如果万一other并没有x或者y方法呢？没有就报异常咯。

如果我们要添加类型的检查，可以这样做：
    
    def +(other)
      Point.new(@x + other.x, @y + other.y)
    rescue
      raise TypeError, "Point addtion is not like a Point Duck Way"
    end

俗语有言：条条大道通罗马。

    def +(other)
      raise TypeError, "Point addtion is not like a Point Duck Way" unless
        other.respond_to? :x and other.respond_to? :y

      Point.new(@x + other.x, @y + other.y)
    end

    def +(other)
      raise TypeError, "Point addtion is not like a Point Duck Way" unless other.is_a? Point

      Point.new(@x + other.x, @y + other.y)
    end

对于Point的实例变量我们可以使用`[]`方法来访问：

    def [](index)
      case index
        when 0, -2: @x
        when 1, -1: @y
        when :x,"x": @x
        when :y,"y": @y
      else
        nil
      end
    end


对于Point类我们也可以定义`each`方法，来遍历实例变量。

    def each
      yield @x
      yield @y
    end

由于Point中的实例变量是有限的，我们只需要yield两次就可以了。调用的时候是需要`p.each {|x| puts x}`即可。

如果类实现了each方法，那么就可以混入Enumerable模块的一些方法，这些方法都是基于each定义的，`include Enumerable`。混入了Enumerable模块，可以写出以下的代码:

  p.all? {|x| x > 0} #return true if all of the elements of Point larger than zero.

在Java中，如果要判断两个实例是否"相等"需要覆写`equals`方法的。那么在Ruby中如何判断两个实例是否相等呢？我们可以定义`==`方法。

    def ==(other)
      if other.is_a? Point
        @x == other.x and @y == other.y
      elsif
        false
      end
    end

在Ruby中`eql?`也可以用来比较对象是否相等，但是它不会自动进行类型的转换，而`==`是会对类型进行自动转换的。
    
    puts 1 == (1.0)  #true FixNum will be converted to Float
    
    puts 1.eql?(1.0)  #false FixNum is not the same as Float

如果想让两个操作在一个类中看起来是一样的，我们可以使用前面学到的`alias`来对方法进行重命名。
    
    alias eql? ==

如果对Point有两种比较方式，一种不严格的坐标相等，一种严格的坐标相等。那么第一种就是`==`方法了。第二种我们可以定义 `eal?`方法来实现。

    def eql?(other)
      if other.instance_of? Point #sub-class instance is not allowed
        @x.eql?(other.x) and @y.eql?(other.y)  #type convert is not allowed
      elsif
        false
      end
    end


__对于Hash值的相等性判断__

Hash类的eql?使用主键进行比较，如果没有定义eql?方法，哈希表会用对象标志对对象进行比较，这意味着如果有一个哈希元素的主键是p，那么只能使用p来访问这个元素，而不能使用q,即使p==q。可变对象不适合做哈希表的主键，让eql?方法保持未定义可以绕过这个问题。

eql?方法用于Hash对象，不能单独定义它，类似于Java，它还需要定义如何计算它的hash值。有一个简单的方法来做到：

    def hash 
      @x.hash + @y.hash
    end

有一个比较通用的hash生成方法，适合于大部分的Ruby类:
  
    def hash
      code = 17
      code = 37 * code + @x.hash
      code = 37 * code + @y.hash
      #the 17 and 37 is from the "Effective Java"
      code
    end

__如何比较两个Point的大小呢？__

如果要比较两个对象的大小，一般要混入Comparable模块，实现`<==>`方法。

    def <==>(other)
      return nil unless other.instance_of Point
      @x ** 2 + @y ** 2 <==> other.x ** 2 + other.y ** 2
    end

Comparable使用`<==>`来定义`==`方法，但是由于Point显式定义了`==`方法，所以Comparable中定义的`==`不会被调用到。

>Enumerable模块定义的若干方法，比如sort, min和max等包含比较的方法只有在被枚举的对象定义了`<==>`才能够正常工作。


在定义一些可变方法的时候，如果会修改原来对象的值，一般对象方法后会有`!`来标志，如果不修改原有对象，一般会返回对象的一个副本。

    def add!(other)
      @x += other.x
      @y += other.y
      self
    end

    def add(other)
      q = self.dup
      q.add!(other)
    end

__可变类Mutable Class__

我们可以使用Struct类来定义可变的类，Struct类是Ruby的内核类，可以用于生成其它的类。在生成的类中定义的实例变量自动具有访问器方法。使用Struct来定义一个新类有两种方式:
    Struct.new("Point", :x, :y)
    Point = Struct.new(:x, :y)

>在上面例子中，第二行代码使用了"命名匿名类"，如果把一个未命名的类对象赋值给一个变量的时候，这个变量名就自动称为该类的名称。上面的类就自动成为了"Point"

使用Struct定义的新类会自动定义getter、setter、[]、[]=、each、each_pair、==、to_s方法。如果需要继续在Point类中添加新的方法，可以直接这样定义:
  
    class Point
      def my
        puts "I am a add-handed method"
      end
    end

这不仅仅限于我们定义的新类，还包括任何其它的类。如果想要取消某些方法，可以使用前面学到的`undef`方法。

    class Point
      undef []=, x=, y=
    end

BTW：所谓的可变与不变是指实例能否被外界的值所改变内部的实例变量。变与不变只是相对的，不变可以变为变的，变的可以变为不变的。

__类方法Class Method__

类方法就是传说中的单例方法，它可以单独调用，也可以使用对象.方法名来调用。定义它的方式有很多种：

    class Point
      #method one
      def Point.sum(*points)
        x, y = 0, 0
        points.each {|p| x += p.x, y += p.y}
        Point.new(x, y)
      end

      #method two
      def self.sum(*points)
        x, y = 0, 0
        points.each {|p| x += p.x, y += p.y}
        Point.new(x, y)
      end

      #method three
      class << self
        def sum(*points)
          x, y = 0, 0
          points.each {|p| x += p.x, y += p.y}
          Point.new(x, y)
        end
      end

    end

    #method four
    class << Point
      def sum(*points)
        x, y = 0, 0
        points.each {|p| x += p.x, y += p.y}
        Point.new(x, y)
      end
    end

__定义类常量__

常量一般都是使用全大写的单词，常量既可以从类中定义，也可以在类外自己动态添加(oh my god, it is so powerful weapon！)

    def Point
      ORIGINAL_POINT = [0,0]
    end

    Point::DYNAMIC = [100, 100]

要访问这些常量一般要加类的限定词，而在类内部对此没有任何要求。

__类变量 Class Variables__

类变量在不同实例之间共享，使用`@@`开头，只在内部可以引用，但是在外部是无法访问到的。类变量可以在类方法中可以被访问，类实例变量却是不可以的。

__类实例变量__

类实例变量是在类方法外定义的变量，但是这些变量是不能被实例方法所访问。不过类实例变量可能会让我们与普通的实例变量所混淆。类实例变量的优于类常量的一个重要特性是在继承现有类时，类实例变量的行为不像类变量那样让人混淆。

下面是分别用类变量和类实例变量写的用于统计Point的代码；
    
    #class variable
    class Point
      @@n = 0 
      @@totalX = 0
      @@totalY = 0

      def initialize(x,y)
        @x,@y = x,y

        @@n += 1
        @@totalX += x
        @@totalY += y
      end

      def self.report
        puts "Number of points created: #@@n"
        puts "Average X coordinate: #{@@totalX.to_f / @@n}"
        puts "Average Y coordinate: #{@@totalY.to_f / @@n}"
      end
    end

    #class instance variable 
    #Because the class instance variable can't be used in the instance method, so define it in the class method
    class Point
      @n = 0 
      @totalX = 0
      @totalY = 0

      def initialize(x,y)
        @x,@y = x,y
      end

      def self.new(x,y)
        @n += 1
        @totalX += x
        @totalY += y

        super
      end

      def self.report
        puts "Number of points created: #@@n"
        puts "Average X coordinate: #{@@totalX.to_f / @@n}"
        puts "Average Y coordinate: #{@@totalY.to_f / @@n}"
      end
    end

因为类实例变量是类对象的实例变量，我们可以使用attr_reader和attr_accessor为它们创建访问器方法。我们可以这样做:
    
    class << self
      attr_accessor :n, :totalX, :totalY
    end

##方法的可见性

Ruby像其它大部分面向对象一样使用public、private和protected来分别表示公开、私有和受保护的。

在ruby中定义的方法默认为public的，但是initialize方法不是，它是私有的。在类外定义的全局方法也是被定义为类的私有方法。

标准的定义类的顺序是这样的；

    class Point
      #public methods
      #...

      #protected methods
      protected

      #private methods
      private

    end

除了常量外，Ruby的变量自动都是私有的，所以我们不能在外面直接访问它，除非有定义访问器方法设定的变量。

也可以在定义了方法后，在类的半部分同义定义某些方法的访问性: `private :x, :y`

如果想要定义工厂类，那么我们一般不让直接使用`类.new`来获取新实例的，所以需要定义new的可访问性，但是由于new是类方法，所以这里需要特殊处理下，即使用`private_class_method`方法。当然如果想要将某个方法变为public的，那么可以使用`public_class_method`方法。

##继承

类在继承的时候不会继承父类的实例变量，这些变量是在方法调用的时候自动生成的，这一点与Java等语言是不一样的。

##对象创建和初始化

Ruby一个new方法**看起来**像这样的:

     def new(*args)
       o = self.allocate #创建类的新对象
       o.initialize(*args) #调用对象的初始化方法，使用传入的参数进行初始化
       o  #返回对象
     end

allocate是Class类的实例方法，被所有的类所继承，它的作用是创建类的一个实例。此方法不能被覆盖，因为Ruby只会调用它的原始版本，所以不会被真的覆盖掉。

initialize方法是一个实例方法，它的作用是为类的实例变量作初始化并赋初值。由于它是一个私有方法，所以我们不能显式地调用它。

>Class类定义了两个名为new的方法，一个是Class#new，它是一个实例方法，另外一个是Class::new，它是一个类方法。Class#new用于创建一个类的新对象，而CLass::new用于创建一个新类。

###创建一个工厂方法

创建工厂方法方法必须不能让外界直接使用new方法，如何做到呢？这就用到了我们上面提到的方法可见性控制的内容了。

    class Point
      def initialize(x,y)
        @x,@y = x,y
      end

      private_class_method :new

      def Point.cartes(x,y)
        new(x,y)
      end

      def Point.polar(r, theta)
        new(r * Math.cos(theta), r * Math.sin(theta))
      end
    end

###dup、clone和initialize_copy

使用dup和clone方法也可以返回一个新对象，它分配一个调用者所属类的实例，然后把调用者的所有实例变量和修改都拷贝到新创建的对象中。clone方法比dup方法拷贝的更彻底，包括对象的单例方法和冻结状态。

如果类定义了一个名为initialize_copy的方法，那么clone和dup方法在拷贝完实例变量后，会执行这个方法，这个方法也是私有方法。

clone和dup方法把实例变量从原始对象拷贝到拷贝对象中时，它们拷贝的是引用而非实际值。也就是说，它们用的是浅拷贝，在修改拷贝对象时，它会修改被拷贝对象的值，所以一般我们在定义一个类时都要定制这两个方法的原因。

    def Point
      attr_accessor :x,:y

      def initialize(x,y)
        @x,@y = x,y
      end
    end

    def Test
      attr_accessor :p

      def initialize(p)
        @p = p
      end
    end

    p = Point.new(0,0)  #x=0,y=0
    t = Test.new(p)   #t.p.x=0, t.p.y=0
    t1 = t.clone
    t2 = t.dup   #the same as the above

    t1.p.x = 1 #p.x=1, t.p.x=1, t2.p.x=1

    t2.p.y = 4 #p.y = 4, t.p.y=4, t1.p.y=4

    t1.p = nil #t.p = p
    t2.p = nil #it have no affect on t

为了防止拷贝对象，我们可以使用`def`；来删除clone和dup方法，也可以将之定义为私有的方法，暴扣new,allocate。

###marshal_dump和marshal_load

创建对象的第三种方式是调用Marshal.load方法来重新生成前面使用Marshal.dump序列化的对象。Marshal.dump方法保存一个对象的类，并递归序列化其中每个实例变量的值。绝大多数对象都可以使用这两个方法进行存储和序列化。

那么什么是序列化呢？序列化是我们将对象的一些状态保存成其它形式，或变量或文件，在需要重新恢复它的状态时，我们可以从序列化的结果进行反向操作，将对象的状态恢复过来。

有些类要修改实现序列化的方式，这样做的原因是为对象状态提供更加紧凑的方式，不去序列化那些缓存易变的数据。修改的方法就是重新定义marshal_dump来定制序列化方式，以及marshal_load来定制反序列化的方式。marshal_load方法被一个使用allocate方法新分配但是未初始化的的对象所调用，它需要一个由marshal_dump返回的可再生的对象拷贝作为参数，然后根据参数对象的状态初始化接收者对象。

  class Point  
    def initialize(*coords)
      @coords = coords  
    end
  
    def marshal_dump
      @coords.pack('w*')
    end

    def marshal_load(s)
      @coords = s.unpack('w*')
    end  
  end
  
  p = Point.new(1,2,3,4)
  s = p.marshal_dump
  t = Point.allocate
  t.marshal_load(s) #t will be the same instance variables as the instance of p

下面的例子展示了如何将对象序列化到文件，并从文件反序列化对象的实例。

    class Logfile
      def initialize(filename)
        @filename = filename
        @io = File.open(@filename,'w')
      end

      def marshal_dump
        log "Begin marshal..."
        @filename  #just dump the filename, and leave the io object alone
      end

      def marshal_load(filename)
        @filename = filename
        @io = File.open(@filename, 'a')
        ``log "Begin ummarshal..."
      end

      def log(msg)
        @io.puts "#{Time.now}: #{msg}"
      end
    end

    logfile = if File.exists?('logfile')
                File.open('logfile') do |file|
                  Marshal.load(file)
                end
              else
                Logfile.new('log.txt')
              end

    ARGV.each do |msg|
      logfile.log msg
    end

    File.open('logfile', 'w') do |file|
      Marshal.dump(logfile, file)
    end

如果对一个类禁用了clone和dup方法，我们可能需要定制序列化方法，通过序列化和反序列化可以很容易实现对象的拷贝，我们就不可以不使用marshal_dump和marshal_load方法，让Marshal.load方法返回对象

##单例类

如果需要设定某个类是单例的，可以将new设置为私有，并且要阻止dup和clone方法不返回新的拷贝。也可以包含`singleton`模块，并在类中include Singleton就可以。这样会定义一个名为instance的方法来返回该类的一个实例。不过需要注意的是，这样定义的类是不能使用带参数的initialize方法。

    require "singleton"

    class PointStats  
      include Singleton

      def initialize
        @n, @totalX, @totalY = 0, 0, 0
      end

      def record(point)
        @n +=1 
        @totalX += point.x
        @totalY += point.y
      end

      def report
        puts "Number of points are: #@n"
      end
    end

在Point类中可以这样定义initialize方法:

      def initialize(x,y)
        @x,@y = x,y
        PointStats.instance.record(self)
      end

想要获取返回的值，可以这样做:
    PointStats.instance.report

#模块Modules

模块与类是很相似的，是方法、常量和变量的命名组，它使用关键字`module`。与类不同的地方是模块不能被实例化，也不能被继承，只能作为命名空间和混入(Mixin)使用。

##模块用于命名空间

一个模块内部是可以相互嵌套的，这样会产生嵌套的命名空间。

    module  Base64  
      DIGITS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'

      class Encoder
        def encoder
          
        end
      end

      class Decode
        def decoder
            
        end
      end

      def Base.help #or self.help
        
      end
    end

在外部使用常量需要Base64::DIGITS，使用某个方法可以使用Base64.help, encoder = Base64::Encoder.new

##Module用于混入

模块的第二个作用是用于混入，Enumerable和Comparable是两个比较常见的模块，前者在混入的类中，如果定义了each方法，那么这个类就会有很多强大的迭代器,如each_with_index,each_with_object。如果Comparable模块被混入，且类定义了`<==>`方法，那么这个类的>、<、=方法就会自动拥有。

    class Point
      include Comparable
    end

上面的include看起来是一个关键字，但实际上它是Module类的一个私有方法,隐式地被self调用。`self.include(Comparable)`。但是在代码中这样书写是错误的，它必须以函数的形式被调用。include方法可以接受多个Module对象进行混入，所以如果一个类定义了each和<==>方法的类可以加入`include Enumerable, Comparable`。

虽然class也是模块，但是类不允许include在另一个类中，include的参数必须是以module进行声明的模块。但是将模块包含在另一个模块中是合法的。

    module Iterable
      include Enumerable

      def each
        loop { yield self.next }
      end
    end

混入一个模块的方式除了使用include外，还可以使用Object.extend方法，它可以将指定模块的实例方法变成接收对象的单键方法。

    countdown = Object.new
    def countdown.each
      yield 1
      yield 3
      yield 2
    end

    countdown.extend(Enumerable) #now the each method becomes a singleton method of Object and have the mothods in Enumerable
    print countdown.sort

##可包含的命名空间模块

前面我们定义的module中的方法都是以混入类的实例方法来调用的，我们可以将混入的方法以类的私有方法来进行调用。在就爱那个方法定义为了实例方法之后，使用`module_function`将这些方法定义为“模块函数”。module_function与public、private等类似，它的主要作用是对给定方法创建类方法的拷贝和将实例方法变为私有的。module_function可以不跟参数，类似与private、public这样的效果，所以如果不想让某些方法成为非模块函数时需要将它定义在module_function的前面。

使用module_function不是出于访问控制的需要，真实目的是让这些方法必须用无接受者的函数风格的调用方式。强制被包含模块的方法以无接收者的方式调用，减少了与真正的实例方法混淆的可能。

    module My 
      def test
        puts "hello world"
      end

      module_function :test
    end

    class Point 
      def my
        test
      end
    end

    p = Point.new
    p.test #Wrong
    p.my #hello world
    My.test #hello world

##加载和请求模块

Ruby程序被分散在多个文件的时候，我们需要将它们“组装”起来，组装的方法就是`require`和 `load`。require和load的作用类似，但是require更加常用，require还可以用于从标准库中加载文件。require还可以加载二进制扩展，如so和dll。load方法要求加载的为包含文件扩展名的完整文件名称，而require只需要传入文件的名字，而不需要后缀。如果一个目录下同时拥有同名的不同后缀的文件，那么require优先加载文件格式的文件，而不是二进制文件。load方法会加载一个路径多次，而require由于会将文件路径展开，所以不会重复，且它把已经加载过的文件名方在全局数组`$"`中。

Ruby的加载路径可以使用`$LOAD_PATH`或`$:`来获取。越是靠前的路径优先被搜索。在ruby 1.9中，load_path数组的元素可以是字符串，也可以是任何实现了`to_path`的类对象。

##执行加载的代码

load和require会立刻执行制定文件的代码，但是这种执行方式与直接调用文件中的代码并不是等价的。

用load和require加载的文件在顶级范围中被执行，而不是在load或require被调用的层级中执行。被夹在的文件可以访问那些加载时已定义的所有全局变量和常量，但是实例变量是不能在文件外被访问到。另外self的值永远是主对象，load和require不会把接收者对象传递给所加载的文件。

load方法在调用时，如果第二个参数值不是nil或false，它会wrap给定的文件到一个匿名模块中，这意味着加载的文件不会影响全局命名空间，它命名的所有常量被放入到这个匿名模块中。这种包裹加载方式作为一种安全措施而存在。

当一个文件被加载到匿名模块中，它亦然可以设置全局变量，而且这些变量也可被加载的代码所使用。


##autoloading Modules

Kernel和Module的autoload方法支持按需惰性加载的机制，它允许使用一个未定义的常量和定义了该常量的包名。在这个常量在第一次被引用的时候，那个注册的包就使用require进行加载。
  
    auload :TCPSocket, "socket"

使用autoload?或Module.autoload?方法可以测试一个常量是否加载一个文件，它们带有一个符号参数。如果这个符号参数加载了一个文件，autoload?方法返回文件名，否则返回nil。


