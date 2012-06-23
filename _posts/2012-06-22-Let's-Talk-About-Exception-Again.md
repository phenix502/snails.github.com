---
layout:       post
title:        "再谈Ruby Exception（笔记）"
description: 
banner: 
categories: 
- ruby learning notes
---

##Throw & Catch
throw和catch是Kernel中的方法，它们定义了一种可以从代码块中，穿过多次的代码级数，与catch一同定义的代码块退出的功能。

Throw和Catch与Break的区别：除了都可以从循环中退出外，前者还可以沿着调用栈向上传播，使一个位于调用方法中的代码块退出。

throw和catch与raise和recue还是不同的。前者只是进行退出，却不抛出异常；后者是抛出异常的。它们有着相似的行为：沿着调用栈向上传递。throw和catch属于控制结构，而不属于异常结构。

	def routine(n)
	  puts n
	  throw :done if n <= 0
	  routine(n-1)
	end
	catch(:done) { routine(3) }


	a = ('a'..'c').to_a
    catch :hello do
      for ele in a do
         throw :hello if ele == 'c'
       end
	end

上面的写法是正确的，如果将catch和throw放到一层，则会报**syntax error, unexpected $end, expecting keyword_end**

如果catch的符号实参与throw的符号实参是不一致的，那么会在运行时报**ArgumentError** [备注1](#comments_1) 异常。 catch和throw可以接受字符串和Symbol，用作符号实参。在处理字符串的时候，内部会自动转换成符号实参。

如果没有throw被调用，那么对应的catch调用就会返回代码块的最后一个值；如果throw被调用，默认是返回nil的。如何不返回nil呢？添加参数。可以对throw添加第二个参数，用作返回值。这样我们可以根据catch的返回值来处理不同的情况，区分正常返回和异常返回。

__在实际情况下，throw和catch并不是很常用，如果在同一个方法中同时包含这2个关键字，那么需要考虑重构代码了。将catch放到单独的方法中，并用return取代throw。__


##Exception
throw并不是抛出异常的关键字，所以throw不是用于处理异常的，只是用于控制的关键字。下面主要设计Exception的处理。

那么异常在神马情况下出现呢？异常出现的情况比较多。比如：使用一个未定义的对象；内存本身已经不足了，而继续申请内存；传递的参数类型不对。这些都会导致异常的产生。

异常发生了之后，异常会传递到最上层或到达异常处理代码，由异常处理代码对异常进行处理或简单粗暴地停止程序的运行，在最外层将异常抛出。

__异常类继承关系图:__

<center>
  <a href="/img/posts/rubyexception_hierarchy.png" class="folio-image-wrapper"><img class="bordered" alt="screenshot" src="/img/posts/ruby_exceptions_hierachy_2.png"style="margin-top:20px; width:50%; height:50%" /></a>
</center>

大部分的异常子类都是继承自StandardError这个异常类，典型的ruby程序会去尝试去处理它们，而其他异常类则一般不会去尝试处理，因为其它异常比较难于恢复。

###异常对象的方法

Exception 类定义了2个返回异常细节的方法:
message和backtrace。message将调用产生的异常信息以人们易于阅读的形式展现出来。message的主要目的是为了给程序员为问题的诊断提供帮助，而不是为了
所谓的简单展示错误信息而已。backtrace方法返回的是一个包含错误调用栈的数组。数组的顺序依次为：发生异常的代码位置;
发生异常的代码调用方法位置;

发生异常的代码调用方法的调用方法位置...以此类推。raise会自动设置调用栈的轨迹，如果要自己创建Exception,可以设置set_backtrace来自定义调用栈轨迹,当然这个方法的参数是一个数组。这个数组是任意的吗？当然不是，它要满足一定的形式：__"filename:LineNo:[in method]"__。
    def a()
      raise "It is an exception"
    end

    def b
      a()
    end

    begin
      b() 
    rescue Exception => e
      puts e.message
      puts e.backtrace.join("\n")
    end
    \_\_END\_\_
    It is an exception
    (irb):32:in `a'
    (irb):36:in `b'
    (irb):40:in `irb_binding'

异常的结果可以使用`$!`访问到，如果异常返回为nil，那么异常为RuntimeError，或者说如果只是简单的raise(msg)，那么抛出的异常是RuntimeError。如果在rescue的时候，指定了异常的变量(比如上面的e)，那么在rescue代码块后是可以使用e来继续访问异常的信息，而`$!`就只会是nil了。

raise的参数还可以是exception，可选第二个参数为异常的信息,第三个为异常调用栈。raise与fail同义，但是我们平常都是使用raise的。

    raise "Failed to create socket"
    raise ArgumentError, "No parameters", caller

rescue本身并不是一个语句，而只是简单的一个方法。recue本身是附着在其它代码块上的，最常见的就是begin..end代码块了。begin..end代码块隔离出一部分代码，并将其中的一部分代码交给rescue进行处理。
  
rescue后面的异常类可以指定多个，recue也可以调用多次：
    begin 
      ...
    rescue RuntimeError,ArgumentError => e，
    recue Exception => e
       ...
    end

retry可以在recue调用完毕后再次重新从代码块开始进行启动。retry适用于值得尝试的异常，比如网咯延迟、服务器宕机；而对于明确不可retry的异常则使用它就没有任何意义了。比如：ArgumentError,ZeroDevisonError。

    require "open-uri"

    tries = 0
    begin
      tries += 1;
      open('http://www.google.com.hk')
    rescue OpenURI::HTTPError => e
      puts e.message
      if (tries < 4)
        sleep(2 ** tries)
        retry
      end
    ensure
      puts "I have tried for #{tries} times."
    end

如果学习过Java，那么我们就会发现与finally对应的一个方法:ensure。rescue用于执行某些资源的释放操作，比如数据库连接的关闭，文件的关闭等等。rescue中的代码可以包含return语句，这会中断异常的传递，也会修改整个方法的返回值；如果ensure中没有显式调用return那么对整个方法的返回值是没有影响的，虽然ruby中最后一行代码的返回值为方法的返回值。

在ensure前面还可以有else方法。它是何许人也？它的作用是如果没有异常发生，那么就else了。也就是如果没有发生异常就执行else里面的句子。它一般不是很常用，除非要处理有异常或没有异常之间的区别处理。else与ensure还有一些区别：else不一定是必然执行的(即使没有异常发生)，ensure是必然执行的。else可以被break、next、return等语句打断跳过。

综上，一个异常的基本结构为:

    begin
      raise 'A test exception.'
    rescue Exception => e
      puts e.message
      puts e.backtrace.inspect
    else
      # other exception
    ensure
      # always executed
    end

   **Attension：**ensure不止是限于begin..end块，还可以用于def method
end、class、module等的定义中。

<div id="comments_1"></div>
+ 备注1: 双飞燕此处是错误的，双飞燕中写的是NameError,根据1.9.3的api，这里应该是ArgumentError。
