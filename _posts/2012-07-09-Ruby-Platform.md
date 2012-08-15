---
layout:       post
title:        "Ruby Platform"
description: 
banner: 
categories: 
- ruby learning notes
---

#Strings

    s = "hello"
    s.concat(" world") #the same as <<
    s.insert(5, " there") #s[5] = " there"
    s.slice(0,5) #the same as s[0,5]
    s.slice!(5,6) #same as s[5,6] = ""
    s.eql?("hello world") #true

    #the length of string 
    s = "hello"
    s.length #5
    s.bytesize #5 中文为每字3byte,ASCII为1byte
    s.size # 5
    s.empty? #false
    "".empty? #true

    s = "hello"
    s.index('l') #2
    s.index(?l) #2
    s.index(/l+/) #2
    s.index('l',3) #the first position after position 3 of l
    s.index('Ruby') #nil
    s.rindex('l') #3
    s.rindex('l',2) #2  index of rightmost l at or before position 2

    s.start_with? "hell" #true
    s.end_with? "bells" #false

    s.include?('ll') #true
    s.include?(?H) #false

    s =~ /[aeiou]{2}/ #nil
    s.match(/[aeiou]/) {|m| m.to_s} #e return first

    "this is it".split #this is it 
    "hello".split('l') #['he','','o']
    "1, 2,3".split(/,\s*/) #1,2,3

    "banana".partition("an") #["b", "an", "ana"] 
    "banana".rpartition("an") #["ban", "an", "a"] 
    "a123b".partition(/\d+/) #["a", "123", "b"] 

    s = "hello"
    s.sub('l', 'L') #replace the first l to L
    s.gsub("l", "L") #replace all the l to L
    s.sub!(/(.)(.)/, '\2\1') #swap first character
    s.sub!(/(.)(.)/, '\\2\\1') #"ehllo" Double backslashes for double quotes

    "hello world".gsub(/\b./) { |match| match.upcase } #Hello World

    s = "hello"
    s.upcase #HELLO
    s.downcase #hello
    s.capitalize #Hello
    s.swapcase #HELLO

    s.casecmp("HELLO") #0 case insensitive comparasion

    s = "hello\r\n"
    s.chomp #"hello"
    s.chomp("o") #hell 
    $/ = ";" #set global separator $/ to semicolon
    "hello;".chomp #hello

    s = "hello\n"
    s.chop #hello  remove line terminator removed, redo will be hell 

    s = "\t hello \n"
    s.strip #hello 
    s.lstrip #hello \n
    s.rstrip #\t hello 

    s = "x"
    s.ljust(3) #"x  "
    s.rjust(3) #"  x"
    s.center(3) #" x "
    s.center(5, '-') #"--x--"
    s.center(7, '-=') #"-=-x-=-"

    s = "A\nB"
    s.each_byte { |b| print b, " " } #65 10 66
    s.each_line { |l| l.chomp } #AB

    s.each_char { |c| print c, " " } #A \n B 

    0.upto(s.length - 1) {|n| print s[n,1], " "}
    "a".succ #b
    "aaz".next #aba
    "a".upto("e") {|c| print c} #abcde

    "hello".reverse #olleh
    "hello\n".dump #escape special character "\"hello\\n\"" 
    "hello\n".inspect #the same as dump

    "hello".tr("aeiou", "AEIOU") #hEllO capitalize vowels
    "head".tr_s("aeiou", "*") #h*d #remove duplicates

    "hello".sum #weak 16-bit checksum 
    "hello".sum(8) #8 bit checksum 
    "hello".crypt("ab") #crypt hello with salt ab。"abl0JrMf6tlhw" 

    "hello".count('aeiou') #2
    "hello".delete('aeiou') #hll
    "hello".squeeze('a-z') #helo remove runs of letters. 
    "hello".count('a-z', '^aeiou') #3 str.count([other_str]+) Each other_str parameter defines a set of characters to count. The intersection of these sets defines the characters to count in str. Any other_str that starts with a caret (^) is negated. The sequence c1–c2 means all characters between c1 and c2.
    "hello".delete('a-z', '^aeiou') #eo str.delete([other_str]+) Returns a copy of str with all characters in the intersection of its arguments deleted. Uses the same rules for building the set of characters as String#count.
    
#正则表达式Regexpression

##正则表达式修饰符

i:匹配时忽略大小写
m:跨行进行模式匹配，换行符被当做普通字符对待。
x:扩展句法，允许在整个表达式中放入空白符和注释。
u,s,e,n: 吧正则表达式解释为Unicode,SJIS,EUC或ASCII，如果没有这样的修饰符，对正则表达式使用源文件的编码方式。

>在正则表达式中`()`,`[]`,`{}`,`.`,`?`,`+`,`*`,`|`,`^`和`$`具有特殊意义，如果在某个匹配模式中包含这些字符的字面量，要用反斜杠进行转义。

>Ruby正则表达式中，字面量可以用`#{}`插入任意的Ruby表达式。

##正则表达式工厂方法

除了直接使用字面量，还可以使用`Regexp.new`或`Regexp.compile`来创建一个正则表达式。

    Regexp.new("Ruby?") #/Ruby?/
    Regexp.compile("Ruby", Regexp::IGNORECASE, "u")

在把一个字符串传递给Regexp的构造方法之前，可以使用Regexp.escape对字符串中的特殊字符进行转义。

    pattern = "[a-z]+"
    suffix = Regexp.escape("()") #treat these characters literally
    r = Regexp.new(pattern + suffix) #/[a-z]+\(\)/

    /R(?i)uby/ #Ignore the case of 'uby'

$~是一个特殊的线程局部和方法局部变量，它在两个并行运行的线程中的值是不同的;使用了=~操作符的方法不会修改调用者方法中的$~变量。

    "hello" =~ /e\w{2}/
    $~.string #hello
    $~.to_s #ell
    $~.pre_match #h
    $~.post_match #o

    if /(?<lang>\w+) (?<ver>\d+(\.\d)+) (?<review>\w+)/ =~ "Ruby 1.9.3 Pre"
      lang #Ruby
      ver #1.9.3
      review #Pre
    end

变量            等价于
$~              RegExp.last_match
$&              RegExp.last_match[0]
$`              RegExp.last_match.pre_match
$'              RegExp.last_match.post_match
$1              RegExp.last_match[1]
$2,etc          RegExp.last_match[2]
$+              RegExp.last_match[-1]

###字符串进行模式匹配

    "ruby123"[/\d+/]  #123
    "ruby123"[/([a-z]+)(\d+)/, 1] #ruby 
    "ruby123"[/([a-z]+)(\d+)/, 2] #123 

slice方法是`[]`方法的同义词方法，不过slice!变体返回值与它相同，并且将从字符串中删除返回的匹配子字符串。

    r = "ruby123"
    r.slice!(/\d+/) #return 123 and r will be ruby 

    pattern = /(['"])([^\1]*)\1/
    text = "He said: 'hello world'"
    text.gsub(pattern) do 
      if $1 == '"'
        "'#$2'"
      else
        "\"#$2\""
      end
    end


