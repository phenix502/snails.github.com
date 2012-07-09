---
layout:       post
title:        "Ruby Reflection And Metaprograming"
description: 
banner: 
categories: 
- ruby learning notes
---

#定义

反射(Reflection)也称为内省(introspection)，表示一个程序可以审视自身的状态和结构。如Ruby可以获得类定义的方法列表，实例变量的值，可以修改自身的状态和结构，动态地增加方法和变量。

元编程(Metaprograming)可以被粗略定义为程序帮助你写程序。在百度百科上的定义为：某类计算机程序编写或操纵其它程序或自身作为它们的数据，或者在变异时完成部分本应在运行时应该完成的工作。 编写元程序的语言被称为元语言，被操作的语言称为目标语言。一门语言同时也是自身的元语言的能力称为反射。

Ruby的反射API大部分定义在Kernel、Module和Object中。反射的本质不是元编程，元编程通常要使用到若干反射机制。

#类型、类和模块Types,Classes And Modules

    module A      
    end

    module B; include A; end;

    class C; include B; end;

    C < B #true
    B < A #true
    C < A #true

    Fixnum < Integer #True
    Integer < Fixnum #false not all integers are fixnums
    Integer < Comparable #true
    String < Numeric #nil, strings are not numbers

    A.ancestors #[A]
    B.ancestors #[B,A]
    C.ancestors #[C, B, A, Object, Kernel, BasicObject] 
    String.ancestors #[String, Comparable, Object, Kernel, BasicObject] 

    C.include?(B) #true
    C.include?(A) #true
    B.include?(A) #true
    A.include?(A) #false
    A.include?(B) #false

    A.included_modules #[]
    B.included_modules #[A]
    C.included_modules #[B,A,Kernel]

>include方法为Module模块的私有方法，所以只能在类或模块的定义中才可以使用。与include方法相关的是公开方法:extend。它可以将制定模块的实例方法作为调用对象的单键方法。

##定义类和模块

Ruby可以动态的创建一个匿名模块或类并赋值给一个常量，常量的名字就称为这个模块或类的名字:

    M = Module.new #module M
    C = Class.new  #class C
    D = Class.new(C) { #D is a subclass of C that includes module M
       include M
    }

    D.to_s #D

##Evaluating Strings and Blocks

Ruby中最强大和最直接的反射特性之一就是eval方法，它可以对字符串进行求值。

    x = 1; eval "x + 1" #2

bingding方法返回一个实例变量的绑定状态。Bingding对象可以作为第二个参数传递给eval方法，指定的字符串就会在绑定的上下文中进行求值。  

    class Object
      def bingdings
        bingding        
      end
    end

    class Test
      def initialize(x)
        @x = x
      end
    end

    t = Test.new(10)
    eval("@x", t.bingdings) #10

###instance_eval和class_eval

Object类定义了instance_eval方法，Module类定义了class_eval方法，这些方法都可以像eval一样对Ruby代码进行求值。但是和eval还是有些区别的。1.这些方法会在指定对象或模块的上下文中对代码进行求值，该对象或模块成为代码求值时self的值。

instance_eval方法为某个对象创建了一个单键方法，而class_eval则定义了一个普通方法。

2.它们可以对代码块进行求值。当参数是代码块而非字符串时，代码块中的代码会在适当的上下文中执行。

    String.class_eval {
      def len
        size
      end
    }

###instance_exec和class_exec

Ruby 1.9中还定义了instance_exec和class_exec两个求值方法，这些方法在接受者对象的上下文中对给定的代码块求值。与前面的class_eval等不同的的地方在于exec可以接收参数，并把参数传递给代码。这样给定的代码块会在给定对象的上下文中执行，并可以获得对象外的参数。

    class Point; end
    x = 10
    p = Point.new
    p.instance_exec(x) { |x| puts x }

###变量和常量

Kernel、Object和Module类定义了很多反射方法列出各种名字，包括所有定义的全局变量、局部变量、实例变量、类变量和常量。

    global_variables #[:$;, :$-F, :$@, :$!, :$SAFE, :$~, :$&, :$`, :$', :$+, :$=, :$KCODE, :$-K, :$,, :$/, :$-0, :$\, :$_, :$stdin, :$stdout, :$stderr, :$>, :$<, :$., :$FILENAME, :$-i, :$*, :$?, :$$, :$:, :$-I, :$LOAD_PATH, :$", :$LOADED_FEATURES, :$VERBOSE, :$-v, :$-w, :$-W, :$DEBUG, :$-d, :$0, :$PROGRAM_NAME, :$-p, :$-l, :$-a, :$binding, :$1, :$2, :$3, :$4, :$5, :$6, :$7, :$8, :$9] 
    x = 1
    local_variables #[:x,:_]

    class Point
      def initialize(x,y)
        @x,@y = x,y
      end

      @@n = 0
      ORIGIN_POINT = Point.new(0,0)
    end

    Point::ORIGIN_POINT.instance_variables #[:@x, :@y] 
    Point.class_variables #[:@@n] 
    Point.constants #[:ORIGIN_POINT] 

###查询、修改、检测变量

Ruby有很多反射方法用于查询、设置和删除变量、类变量、常量。
    x = 1
    varname = "x"

    eval(varname) #1
    eval("varname = '$g'") #$g
    eval(varname) #nil,because varname now is $g,so its eval is nil
    eval("#{varname} = x") #1

eval可以任何对象的实例变量、任何类或模块的类变量和常量进行查询、设置和存在性检测操作:

    o = Object.new
    o.instance_variable_set(:@x, 0) #require @ prefix
    o.instance_variable_get(:@x) #0
    o.instance_variable_defined?(:@x) #true

    Object.class_variable_set(:@@x, 1)
    Object.class_variable_get(:@@x) #1
    Object.class_variable_defined?(:@xx)

    Math.const_set(:EPI, Math::PI * Math::E) #8.539734222673566 
    Math.const_get(:EPI)
    Math.const_defined?(:EPI)

在Ruby 1.9中如果把false作为const_get和const_defined?的第二个参数，则对应的变量是在当前类或模块中进行查找，不会去寻找父类或模块的常量。

Object和Module对象中的一些私有方法被用来取消实例变量、类变量和常量的定义，它们都返回被删除变量或常量的值。因为这些方法都是私有的，所以不能直接用在对象、类和模块上，只能通过`eval`或`send`方法完成。

    o.instance_eval { remove_instance_variable :@x }
    String.class_eval { remove_class_variable :@@x }
    Math.send :remove_const, :EPI

如果一个模块定义了const_missing方法，当找不到该变量的时候，该方法就会被调用。

    def Symbol.const_missing(name)
      name #return the name of the missing Const
    end

    Symbol::TEST #:TEST

###Methods

    o = "hello world"
    o.methods #all the public methods of String
    o.public_methods #the same as the above
    o.public_methods(false) #the inherited methods
    o.protected_methods #all the protected methods
    o.private_methods #all the private Methods
    o.private_methods(false) #inherited private methods
    def o.single; 1; end
    o.singleton_methods #:single

    #Query
    String.instance_methods == "s".public_methods #true
    String.instance_methods(false) == "s".public_methods(false) #true
    String.public_instance_methods == String.instance_methods #true
    String.protected_instance_methods #[]
    String.private_instance_methods(false) #[:initialize, :initialize_copy]

    #Check
    String.public_method_defined? :reverse #true
    String.protected_method_defined? :reverse #false
    String.private_method_defined? :initialize #true
    String.method_defined? :upcase #true

Module.method_defined?方法用于检测给定的名称是否用于定义了一个公开或保护的方法，它与Object.respond_to?方法的功能基本类似。在Ruby 1.9中可以制定该方法的第二个参数为false表示不考虑继承方法。

要查询某个特定名称的方法，可以在任意对象上调用`method`方法，或者在任意模块上调用`instance_method`方法。前者返回一个绑定于接收者的可调用Method对象，后者返回一个UnboundedMethod对象。在Ruby 1.9中，可以使用`public_method`和`public_instance_method`来获取对象的公开方法。

    "o".method(:reverse) #<Method: String#reverse>
    String.instance_method(:reverse) #<UnboundMethod: String#reverse> 

通常调用一个有名字的方法使用`send`方法:
    
    "o".send :upcase #O
    Math.send(:sin, Math::PI / 2) #1

send在接收者对象上调用名为第一个参数的方法，后面的其他参数作为调用方法的参数。方法名`send`来自于面向对象编程的术语，调用一个方法，即向对象"发送"一个消息。

>send方法可以调用一个对象的任意有名方法，也包括私有或保护方法。

同样的在Ruby 1.9中定义了一个以`public`开头的方法，使send只调用公开的方法

上面说到了如何调用一个方法，那么如何在一个类中添加、取消或者别名一个方法呢？

如果想要定义一个新的实例方法，可以使用`define_method`方法，它是Module的实例方法，第一二参数是新方法的名字，方法体要使用Method对象或代码块作为第二个参数。define_method是一个私有方法，所以必须在类或模块中调用它。

    def add_method(c, m, &b)
      c.class_eval{
        define_method(m, &b)
      }
    end

    add_method(String, :greet) { "Hello, " + self }
    "world".greet #hello world

    #define a class method 
    def add_class_method(c, m, &b)
      eigenclass = class << c; self; end
      eigenclass.class_eval {

        define_method(m, &b)
      }
    end

    add_class_method(String, :greet) { |name| "hello " + name }

    String.greet("world") #hello world

在Ruby 1.9中，可以直接使用`define_singleton_method`方法来定义一个类方法:

      String.define_singleton_method(:greet) { |name| "hello " + name }

>define_method的缺点是不能定义方法体使用代码块的方法。

如果想动态创建一个接受代码块的方法，需要联合使用def和class_eval方法。如果要创建的方法有足够的动态性，可能无法把一个代码块传给class_eval方法，这时则需要用一个字符串表示要定义的方法，然后对它进行求值。

如果需要对方法创建同义或别名方法，可以使用`alias`语句:
    
    alias plus +

在动态编程时，有时需要使用`alias_method`方法。alias_method经常用于创建别名链。

    def backup(c, m, prefix="orig")
      n = :"#{prefix}_#{m}"
      c.class_eval {
         alias_method n, m
      }
    end

    backup(String, upcase)
    "o".orig_upcase #O

>undef方法用于取消一个方法的定义，但是这仅适用于硬编码标识符表示名字的方法。如果需要动态删除一个方法，可以使用`remove_method`或`undef_method`。它们都是Module类的私有方法，remove_method用于删除当前类中的方法定义，如果在父类中包含该方法，那么该方法会被继续继承下来，而undef_method则会不允许再次在实例上调用该方法。

method_missing方法通常是在Ruby无法找到某个方法时所调用的方法，默认情况下method_missing只是抛出NoMethodError异常，如果不对异常进行捕获，那么整个程序就会退出。

    class Hash
      def method_missing(key, *args)
        text = key.to_s

        if text[-1,1] == "=" #if key ends with = set a value
          self[text.chop.to_sym] = args[0]
        else
          self[key]
        end
      end
    end

    h = {}
    h.one = 1 #the same as h[:one] = 1
    puts h.one #1

##Hooks

Module、Class和Object类实现了若干回调方法，这些方法也被称为钩子方法。钩子方法并非默认定义的，在特定事件发生时会被调用。这样我们可以在定义子类、包含模块或定义方法时可以扩展Ruby的行为。它们通常以`ed`结尾。

在一个类被继承的时候，通常会回调它的`inherited`方法，当模块被包含时，会调用它的`included`方法：

    module Final
      def self.included(c)
        c.instance_eval do
          def inherited(sub)
            raise Exception, "Attempt to create subclass #{sub} of Final class #{self}"
          end
        end
      end
    end

如果一个模块定义了一个名为extended的类方法，那么它将在一个对象扩展该模块时被调用。

除了有跟踪包含类和模块的钩子方法，还有用于跟踪类和模块的方法的钩子方法，以及跟踪任意单键方法的钩子方法。如果为人以类或模块定义了一个名为`method_added`的方法，它将在该类或模块定义一个实例方法时被调用。

    def String.method_added(name)
      puts "New instance method #{name} added to String."
    end

method_added方法会被子类所继承。但是因为这个钩子方法没有任何类信息的参数，所以在添加某个方法时，是无法直到是定义method_added方法所在的类添加的还是子类添加的。解决这个问题的方法是为定义method_added的类的同时定义inherited方法，然后在inherited方法为每个子类定义一个method_added方法。

     class Strict 
      def self.method_added(name)
        STDERR.puts "Warning: #{name} method added"
        remove_method name
      end
    end
    
    class Strict
      def a
      end
    end  #Warning: a method added

    s = Strict.new
    s.a #NoMethodError: undefined method `a'

#ObjectSpace和GC

ObjectSpace模块定义了一组方便的的低级方法，对调试和元编程有所帮助。最重要的方法为:each_object，它可以迭代解释器知道的每个对象。

    ObjectSpace.each_object(Class) { |c| puts c } #in ruby 1.9 it will about 383 objects 

`_id2ref`是Object.object_id的反向方法，它的参数是一个对象ID，它返回相对应的对象；如果没有对应的对象，则抛出一个RangeError异常。

ObjectSpace.define_finalizer可以注册一个Proc对象或一个代码块，它们在给定对象被垃圾收集时调用。任何用于终结(finalize)对象的值必须能够被finalizer块获得，这样它们无需通过被终结对象而使用。它的反向方法为undefine_finalizer。

ObjectSpace.garbage_collect强制让Ruby运行垃圾收集器。垃圾收集功能也可以通过GC模块获得。GC.start是ObjectSpace.garbage_collect的同义方法，可以通过GC.disable方法临时关闭垃圾收集，用GC.enable使之再次生效。


    class Object
      def trace(name="", stream=STDERR)
        TraceObject.new(self, name, stream)
      end
    end

    class TraceObject
      instance_methods.each do |m|
        m = m.to_sym
        next if m == :object_id || m == :__id__ || m == :__send__
        undef_method m
      end

      def initialize(o, name, stream)
        @o = o
        @n = name
        @trace = stream
      end

      def method_missing(*args, &block)
        m = args.shift #the method name
        begin
          arglist = args.map { |a| a.inspect }.join(', ')
          @trace << "Invoking: #{@n}.#{m}(#{arglist}) at #{caller[0]}\n"
          r = @o.send m, *args, &block
          @trace << "Returning: #{r.inspect} from #{@n}.#{m} to #{caller[0]}\n"
          r
        rescue Exception => e 
          @trace << "Rasing: #{e.class}:#{e} from #{@n}.#{m}\n"
          raise
        end
      end

      def __delegate
        @o
      end
    end

##动态创建方法
    
    #Method 1: class_eval
    class Module
      private 
      def readonly(*syms)
        return if syms.size == 0
        code = ""
        syms.each do |s|
          code << "def #[s]; @#{s}; end\n"
        end
        class_eval code
      end

      def readwrite(*syms)
        return if syms.size == 0
        code = ""
        syms.each do |s|
          code << "def #{s}; @#{s}; end\n"
          code << "def #{s}=(value); @#{s} = value; end\n"
        end

        class_eval code
      end
    end

    #Method 2:define_method
    class Module
      def attributes(hash)
        hash.each_pair do |sym, default|
          getter = sym 
          setter = :"#{sym}"
          variable = :"@#{sym}"

          define_method getter do
            if instance_variable_defined? variable
              instance_variable_get variable
            else
              default
            end
          end

          define_method setter do |value|
            instance_variable_set variable, value
          end
        end
      end

      def class_attrs(hash)
        eigenclass = class << self; self; end
        eigenclass.class_eval { attributes(Hash) }
      end

      private :attributes, :class_attrs
    end

##Alias Chaining

    module ClassTrace 
      T = []

      if x = ARGV.index("--traceout")
        OUT = File.open(ARGV[x+1], "w")
        ARGV[x+2] = nil
      else
        OUT = STDERR
      end
    end

    alias ori_load load
    alias ori_require require

    def require(file) 
      ClassTrace::T << [file, caller[0]]
      ori_require file
    end

    def load(*args)
      ClassTrace::T << [args[0], caller[0]]
      ori_load(*args)
    end

    def Object.inherited(c)
      ClassTrace::T << [c, caller[0]]
    end

    at_exit {
      o = ClassTrace::OUT
      o.puts "="*60
      o.puts "Files loaded and Classes Defined:"
      o.puts "="*60

      ClassTrace::T.each do |what, where|
        if what.is_a? Class 
          o.puts "Defined:#{what} at #{where}"
        else
          o.puts "Loaded:#{what} at #{where}"
        end
      end
    }

  

    #Tracing Methods
    class Object
      def trace!(*methods)
        @_traced = @_traced || []
        methods = public_methods(false) if methods.size == 0
        methods.map! { |m| m.to_sym }
        methods -= @_traced
        return if methods.empty?
        @_traced |= methods 

        SDTERR << "Tracing #{methods.join(', ')} on #{object_id}\n"

        eigenclass = class << self; self; end

        methods.each do |m|
          eigenclass.class_eval %Q{
             def #{m}(*args, &block)
               begin
                 STDERR << "Entering #{m}(\#{args.join(', ')})\n"
                 result = super
                 STDERR << Exiting #{m} with \#{result}\n"
                 result
             rescue 
               STDERR << "Aboring #{m}: \#{$!.class}, \#{$1.message}"
               raise
             end
          }
        end
      end

      def untrace!(*methods)
        if methods.size == 0
          methods = @_traced
          STDERR << "Untracing all methods on #{object_id}"
        else
          methods.map! { |m| m.to_sym }
          methods &= @_traced
          STDERR << "Untracing #{methods.join(', ')} on #{object_id}\n"
        end

        @_traced -= methods
        (class << self; self; end).class_eval do 
          methods.each do |m|
            remove_method m
          end
        end

        if @_traced.empty?
          remove_instance_variable @_traced
        end
      end
    end

##DSL(Domain-Specific Language)

领域特定语言是对Ruby句法或API的扩展，可以用更自然的方式处理问题或表现数据。

    class XML
      def initialize(out)
        @out = out 
      end

      def context(text)
        @out << text.to_s
      end

      def comment(text)
        @out << "<!-- #{text} -->"
        nil
      end

      def tag(tagname, attributes={})
        @out << "<#{tagname}"

        attributes.each {|attr,value| @out << " #{attr}=`#{value}`"}

        if block_given?
          @out << '>'
          content = yield
          if content 
            @out << content.to_s
          end
          @out << "</#{tagname}>"
        else
          @out << '/>'
        end
        nil 
      end

      alias method_missing tag

      def self.generate(out, &block)
        XML.new(out).instance_eval(&block)
      end
    end

    pagetitle = "Test Page for XML.generate"
    XML.generate(STDOUT) do
      html do
        head do
          title { pagetitle }
          comment "This is a test"
        end
        body do
          ul :type => "sequare" do
            li { Time.now }
            li { RUBY_VERSION }
          end
        end
      end
    end


    #Example 2

    class XMLGrammar
      def initialize(out)
        @out = out 
      end
      
      def self.generate(out, &block)
        new(out).instance_eval(&block)
      end

      def self.element(tagname, attributes={})
        @allowed_attributes ||= {}
        @allowed_attributes[tagname] = attributes

        class_eval %Q{
           def #{tagname}(attributes={}, &block)  
             tag(:#{tagname}, attributes, &block)
           end
        }
      end

      OPT = :opt
      REQ = :req 
      BOOL = :bool


      def self.allowed_attributes
        @allowed_attributes
      end

      def content(text)
        @out << text.to_s
        nil
      end

      def comment(text)
        @out << "<!--#{text}-->"
        nil
      end

      def tag(tagname, attributes={})
        @out << "<#{tagname}"
        allowed = self.class.allowed_attributes[tagname]
        
        attributes.each_pair do |key, value|
          raise "unknown attributes:#{key}" unless allowed.include?(key)
          @out << " #{key}='#{value}'"
        end

        allowed.each_pair do |key, value|
          next if attributes.has_key? key
          if (value == REQ)
            raise "required attributes '#{key}' missing in <#{tagname}>"
          elsif value.is_a? String
            @out << " #{key}='#{value}'"
          end
        end

        if block_given?
          @out << '>'
          content = yield
          if content
            @out << content.to_s 
          end
          @out << "</#{tagname}>"
        else
          @out << '/>'
        end

        nil
      end    
    end

    class HTMLForm < XMLGrammar
      element :form, :action => REQ,
                     :method => "GET",
                     :enctype => "application/x-www-form-urlencoded",
                     :name => OPT 
      element :input, :type => "text", :name => OPT, :value => OPT,
                      :maxlength => OPT, :size => OPT, :src => OPT,
                      :checked => BOOL, :disabled => BOOL, :readonly => BOOL
      element :textarea, :rows => REQ, :cols => REQ, :name => OPT,
                         :disabled => BOOL, :readonly => BOOL
      element :button, :name => OPT, :value => OPT,
                       :type => "submit", :disabled => OPT
    end

    HTMLForm.generate(STDOUT) do
      comment "This is a simple HTML form"
      form :name => "registration",
           :action => "http://test.cig" do
        content "Name:"
        input :name => "name"
        content "Address:"
        textarea :name => "address", :rows => 6, :cols => 40 do
          "Please enter your mailing address here"
        end
        button { "Submit" }
      end
    end
