---
title: 批处理删除目录下的文件
---

有时候VS进程会锁住生成的程序集，导致无法复制文件等错误，需要手工删除。编辑个批处理来点干脆点的，把所有bin子目录的文件全部删除。
```dos
@echo This bat file clear all files in bin directory including subdirectory
@(dir /s/ad/b bin) > kill.txt
@for /f “delims=.”%%d in (kill.txt) do del /q/s %%d\*.*
@del kill.txt
```
REM delims=.表示用.来分割，默认是空格。因为目录名有可能中间带空格，采用默认选项的话会导致得到的%%d不完整。如目录名"stand config"，最终就会变成执行del /q/s stand\*.*
```dos
dir /s/ad/b bin > kill.txt
```
把所有子目录下的bin目录输入到kill.txt中。/s表示含子目录，/ad表示目录，/b表示只输出目录名

使用Ruby来完成此功能 
使用递归来查找所有的bin目录，并递归删除所有目录下的文件。Ruby中删除不为空的目录会报异常。比较起来，还是命令简洁些
```ruby
#delete all subdirs and files
def delete_star(d)
  Dir.foreach(d) do |e|
    next if[".",".."].include? e
    name = d + File::Separator + e
    if File::directory?(name)
      delete_star(name)
    else
      File.delete(name)
      puts "delete #{name}"
    end
  end
end

#find all bin directiory
def findbin(d)
  Dir.foreach(d) do |e|
    next if[".",".."].include? e
    name = d + File::Separator + e
    next if not File::directory?(name)
    if e== "bin"
      delete_star(name)
    else
      findbin(name)
    end
  end
end

findbin(".")
```