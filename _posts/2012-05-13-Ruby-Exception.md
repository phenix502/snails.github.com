---
layout:       post
title:        "Ruby Programing 1.9学习笔记"
description: 
banner: 
categories: 
- ruby learning notes
---

##异常处理
真实世界充满了各种不确定性因素，我们的应对的时候都需要考虑到，有所应对。如果没有考虑到这些，后果有时候蛮严重的。我们的程序也是一样的，如果没有处理好异常，那么我们的程序会很失败，很难看。

##传统方式

学过C语言的筒子们，在操作文件的时候，函数通常会返回一个叫做句柄的东东，通过这个东东来判断我们操作的成功与否，如果出错了，错误的原因又是神马（通过返回代码）。

不同的函数，返回的错误代码是不一样的。而“异常”的出现将错误进行了一定程度上的统一——异常(Exception)类,实现了对象化，异常对象将错误信息进行自动打包，然后返回给能够处理它的Runtime Environment。

##异常类继承图
异常类的继承关系图：
<center>
  <a href="/img/posts/rubyexception_hierarchy.png" class="folio-image-wrapper"><img class="bordered" alt="screenshot" src="/img/posts/rubyexception_hierarchy.png" style="margin-top:20px; width:50%; height:50%" /></a>
</center>

##定义异常
如何定义一个异常呢？异常的父类是：`Exception`，只要自己的类继承自它或者它的子类就可以。

例如：
	class MyException < RuntimeError
	  attr :my_error
	  def initialize(error)
        @my_error = error
	  end
    end

在调用的时候：
	def read_data(file_stream)
      if file_stram.nil?
        raise MyException.new(true), "The file stream is not assigned."
      end
    end

##处理异常

	require "open-uri"

	page = "podcasts"
	file_name = "#{page}.html"
	web_page = open("http://pragprog.com/#{page}")
	output = File.open(file_name, "w")
	begin
 	 while line = web_page.gets
   		 output.puts line
  	 end
  	 output.close
	rescue Exception
  	  STDERR.puts "Failed to download the page content."
      output.close
  	  File.delete(file_name)
  	  raise
	end

在抛出异常和处理异常的时候，Ruby把异常对象放到一个叫做`“$!”`的全局变量中。上面代码中的raise将"$!"抛向上一层处理异常的代码。

在Ruby中，我们可以多次从异常中rescue，执行的顺序和case语句是雷同的。如果异常没有匹配rescue，那么异常会寻找调用者或调用者的调用者中的异常处理对象。

例如：
	begin
       expresion1 …
    rescue SystaxError, NameError => error_1
      puts "This is #{error_1}"
    rescue StandardError => error_2
      puts "This is #{error_2}"
    end

在其它语言（Java、Delphi等）有一个关键字“finally”,在遇到异常的时候，它里面的代码依然会执行。这部分的代码作用一般是执行一些清理操作，比如释放一些资源。在Ruby中也有类似的关键字："ensure"。

例如：
	begin
      expression…
    rescue 
      handle error…
    ensure
      clean up
    end

上面的rescue如果没有参数，默认为StandardError。
在上面的代码中的ensure上面，还可以插入一段else代码，它的作用是：如果没有异常执行的操作。不过用途不大。

在rescue中，还可以执行retry方法，它的用途是：重新执行整个being/end语句块。一般在rescue中会有些修正的代码，以便程序可以再次执行。

例如：

	@eflag = true
	begin
  	if @eflag then \#then可选
      do_sth
  	else
   	  do_sth2
 	end

	rescue StandardError
  	  if @eflag 
        @eflag = false
        retry
 	  else
        raise
  	  end
 	end

##抛出异常
	raise
	raise 'hello world'
	railse BadArgument, "Arguments Error", caller

第一种形式只是简单地将当前的异常（如果没有异常，那么抛出的为RuntimeError），一般应用于在继续传递前，拦截某个异常。

第二种形式会新建一个RuntimeError异常，并将后面的信息放到异常信息中，异常会显示调用栈。

第三种形式是在第二种的基础上，添加了异常的类，和调用栈的trace。caller一般是Kernel.caller方法。caller可以跟一个数组，默认返回当前的调用栈信息。

如果想要调用caller的caller异常，可以这样操作：
	raise $!.class, $!.message, $!.backtrace[2..(-1)]


##Catch和Throw
异常的raise和rescue在程序出错时终止执行，但是有时候使用catch和throw来从nested construct跳出会更适用。

例如：
	word_list = File.open("wordlist") 
	  word_in_error = catch(:done) do
  	    result = []
  	    while line = word_list.gets
   	    word = line.chomp
        throw(:done, word) unless word =~ /^\w+$/ #word可选
        result << word 
  	  end
      puts result.reverse
	end

    if word_in_error puts "Failed: '#{word_in_error}' found, but a word was expected"
    end

上述代码，如果在打开的文件中，如果内容有无效的单词，就会抛出一个Symble（字符串也是可以的）,并将这个单词返回。在后面进行输出。

另外catch不一定要放在throw的静态作用域之内。

例如：
	def prompt_and_get(prompt)
	  print prompt
	  res = readline.chomp 
	  throw :quit_requested if res == "!"
	  res
	end
	catch :quit_requested do
	name = prompt_and_get("Name: ") 
	age = prompt_and_get("Age: ")
	sex = prompt_and_get("Sex: ")
	# ..



