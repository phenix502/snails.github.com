---
layout:       post
title:        "使用Spreadsheet读取和修改Excel文件"
description: 
banner: 
categories: 
- ruby 
---

#Spreadsheet是神马

Spreadsheet是一个Ruby实现的gem，它可以使我们很方便的使用它对excel进行操作。

#如何安装

    gem install spreadsheet #你懂得

#简单使用

Spreadsheet基本使用的包括以下几部分(C表示类,M表示模块):

* Column: (C)用来指定列的格式化和提供了以列的形式遍历所有单元格的方法。
* Datatypes: (M)定义了可以将参数转换为Spreadsheet常用的属性的方法(boolean,colors,enum)，
* Encoding: (C)定义了编码转换的方法，基本用不到。
* Row: (C)定义了对行为单位对Excel进行处理的方法,它是Array的子类，所以很多Array的方法它都可以使用。
* Workbook: (C)主要来操作Excel的标签页，每个标签页就是一个Workbook.
* Worksheet: (C)一个新的实例就是一个Excel。
* Format: (C)定义Cell的格式的。
    :font=>Spreadsheet::Font :number_format=>"GENERAL", :rotation=>0, :pattern=>1, :bottom_color=>:builtin_black, :top_color=>:builtin_black, :left_color=>:builtin_black, :right_color=>:builtin_black, :diagonal_color=>:builtin_black, :pattern_fg_color=>:yellow, :pattern_bg_color=>:border, :regexes=>{:date=>>/[YMD]/, :date_or_time=>>/[hmsYMD]/, :datetime=>>/([YMD].*[HS])|([HS].*[YMD])/, :time=>>/[hms]/}, :used_merge=>0, :horizontal_align=>:center, :text_wrap=>false, :vertical_align=>:bottom, :rotation_stacked=>false, :indent_level=>0, :shrink=>false, :text_direction=>:context, :left=>:none, :right=>:none, :top=>:none, :bottom=>:none, :cross_down=>false, :cross_up=>false
* Font: (C)看名字就知道这个字体。
    :name=>"Arial", :color=>:text, :weight=>700, :size=>14, :italic=>false, :strikeout=>false, :outline=>false, :shadow=>false, :escapement=>:normal, :underline=>:none, :family=>:swiss, :encoding=>:iso_latin1

#操作一个Spreadsheet
    # 引入spreadsheet插件
    require "spreadsheet"

    # 声明Spreadsheet处理Excel文件组时的编码
    Spreadsheet.client_encoding = "UTF-8"

    # 创建一个Spreadsheet对象，它相当于Excel文件
    book = Spreadsheet::Workbook.new
    # 创建Excel文件中的一个表格，并命名为 "Test Excel"
    sheet1 = book.create_worksheet :name => "Test Excel"

    # 设置一个Excel文件的格式
    default_format = Spreadsheet::Format.new(:weight => :bold,#字体加粗
                                 :size => 14, 
                                 :horizontal_align: => :merge, #表格合并
                                 :color=>"red", 
                                 :border=>1, 
                                 :border_color=>"black",
                                 :pattern => 1 ,
                                 :pattern_fg_color => "yellow" )#这里需要注意，如果pattern不手动处理，会导致pattern_fg_color无实际效果

    data = "测试"

    # 指定一个在表格中的第一行对象
    test_row = sheet1.row(0)

    # 为第一行的前5个列指定格式
    5.times do |i|
      test_row.set_format(i, default_format)
    end

    # 为第一行的第一列指定值
    test_row[0] = data

    # 将创建的Spreadsheet对象写入文件，形成电子表格
    book.write 'book2.xls'

如果需要合并某些列，建议使用merge_cells(begin_row,begin_col,end_row,end_col)。i

#格式化时间

当设置某个cell为时间时，Spreadsheet将会尝试将cell的number_format到一个相关的值。如果自己定义了时间的格式，Spreadsheet将会使用程序员定义的。如果一个Cell已经被格式化，Spreadsheet将会保留原有的格式。

    row[4] = Date.new 1975, 8, 21
    # -> assigns the builtin Date-Format: 'M/D/YY'
    book.add_format Format.new(:number_format => 'DD.MM.YYYY hh:mm:ss')
    row[5] = DateTime.new 2008, 10, 12, 11, 59
    # -> assigns the added DateTime-Format: 'DD.MM.YYYY hh:mm:ss'
    row.set_format 6, Format.new(:number_format => 'D-MMM-YYYY')
    row[6] = Time.new 2008, 10, 12
    # -> the Format of cell 6 is left unchanged.

#参考资料:
1. http://blog.csdn.net/xianqiang1/article/details/7298389
2. http://stackoverflow.com/questions/7730112/ruby-spreadsheet-row-background-color
3.http://stackoverflow.com/questions/11603216/merging-cells-with-ruby-gem-spreadsheet
4. Spreadsheet Guide
