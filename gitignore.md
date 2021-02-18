# gitignore
## 用途
.gitignore用来忽略git项目中一些文件，使得这些文件不被git识别和跟踪；

简单的说就是在.gitignore添加了某个文件之后，`git status`就不会再看到这些文件了。

## 语法
* 空行或是以（#）开头的行即注释行将被忽略。
* 可以在前面添加正斜杠（/）来避免递归
* 可以在后面添加正斜杠（/）来忽略文件夹
* 可以使用（!）来否定忽略
* 可以使用标准的 glob 模式匹配，所谓的 glob 模式是指 shell 所使用的简化了的正则表达式：
  * 星号（*）匹配零个或多个任意字符
  * [abc]匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）
  * 如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）
  * 问号（?）只匹配一个任意字符
  * 使用两个星号（\*\*) 表示匹配任意中间目录，比如a/**/z可以匹配 a/z, a/b/z 或 a/b/c/z等

## 例子
```
# 忽略 .a 文件，是所有目录的下的 .a 文件
*.a
# 否定忽略 lib.a，尽管已经在前面忽略了 .a 文件
!lib.a
# 仅在当前目录下忽略 TODO 文件，但不包括子目录下的 subdir/TODO
/TODO
# 忽略 build/ 文件夹下的所有文件
build/
# 忽略 doc/notes.txt, 不包括 doc/server/arch.txt
doc/*.txt
# 忽略所有在 doc/ 目录里的 .pdf 文件 
doc/**/*.pdf
```

## 常用的模版
https://github.com/github/gitignore

## 参考
https://www.jianshu.com/p/ea6341224e89  
https://blog.csdn.net/le_17_4_6/article/details/92789993