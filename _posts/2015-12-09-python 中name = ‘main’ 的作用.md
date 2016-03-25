---
layout: post
tile:  "python 中name = ‘main’ 的作用"
date:  2015-12-09 16:38:25
categories: python 
excerpt: python 中name = ‘main’ 的作用
---

* content
{:toc}

#python 中__name__ = '__main__' 的作用


[TOC]

##1、先看看_name_的定义：

1. 如果模块是被导入，__name__的值为模块名字
2. 如果模块是被直接执行，__name__的值为__main__


##2、再看看 __name__ = '__main__' 
很多新手刚开始学习python的时候经常会看到python 中__name__ = '__main__' 这样的代码，可能很多新手一开始学习的时候都比较疑惑，python 中__name__ = '__main__' 的作用，到底干嘛的？

有句话经典的概括了这段代码的意义：

“Make a script both importable and executable”

意思就是说让你写的脚本模块既可以导入到别的模块中用，另外该模块自己也可执行。

这句话，可能一开始听的还不是很懂。下面举例说明：

先写一个模块：
	
	#mymodule.py
	def main():
	  print "we are in %s"%__name__
	if __name__ == '__main__':
	  main()

这个函数定义了一个main函数，我们执行一下该py文件发现结果是打印出”we are in __main__“,说明我们的if语句中的内容被执行了，调用了main()：

但是如果我们从另我一个模块导入该模块，并调用一次main()函数会是怎样的结果呢？
	
	#anothermodle.py
	from module import main
	main()

其执行的结果是：we are in mymodule

但是没有显示”we are in __main__“,也就是说模块__name__ = '__main__' 下面的函数没有执行。

这样既可以让“模块”文件运行，也可以被其他模块引入，而且不会执行函数2次。这才是关键。

##3、总结

如果我们是直接执行某个.py文件的时候，该文件中那么”__name__ == '__main__'“是True,但是我们如果从另外一个.py文件通过import导入该文件的时候，这时__name__的值就是我们这个py文件的名字而不是__main__。

这个功能还有一个用处：调试代码的时候，在”if __name__ == '__main__'“中加入一些我们的调试代码，我们可以让外部模块调用的时候不执行我们的调试代码，但是如果我们想排查问题的时候，直接执行该模块文件，调试代码能够正常运行！
