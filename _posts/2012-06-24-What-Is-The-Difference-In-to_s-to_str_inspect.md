---
layout:       post
title:        "The Difference Of XX-like Methods In Ruby"
description: 
banner: 
categories: 
- ruby learning notes
---

今日打开ruby-china发现Hooopo分享的一篇文章，感觉非常好，故记录之。
#to_s和inspect的区别
class David
  def to_s
    "to_s"
  end

  def inspect
    "inspect"
  end
end

david = David.new  #inspect
puts david #to_s
print david #to_s
p david #inspect

__结论:__ 

__1. puts obj => puts obj.to_s__

__2. p obj => puts obj.inspect__

#to_s和to_str的区别

to_s和to_str在大部分时候是相同的，几乎每个对象都有to_s方法，(why?因为所有对象都继承自Object类)，但是不是每个对象都有to_str方法，这个方法只有在对象有string-like的行为时才定义。

但是并不是所有和字符串相关的方法都会调用to_str：

    class David
      def to_str
         "to_str"
      end

       def to_s
         "to_s"
       end
    end
    
    david = David.new #to_s
    "hello, #{david}" #hello,to_s
    ['hello', david].join(" ") #heloo to_str
    "hello " + david #hello to_str
    File.join("hello", david) #hello/to_str

**根据上面的结果得出：在字符串内插和inspect的时候会调用to_s，而在Array#join,File#join,String#+的时候优先调用to_str**

其它类XX-like的方法还有to_i vs to_int; to_a vs to_ary 
