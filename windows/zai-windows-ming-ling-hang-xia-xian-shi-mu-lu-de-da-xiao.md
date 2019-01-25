# 在 Windows 命令行下显示目录的大小



{% hint style="info" %}
转载自 [在 Windows 命令行下显示目录的大小 - aneasystone's blog](https://www.aneasystone.com/archives/2015/08/folder-size-in-windows-command-line.html)
{% endhint %}

我们知道在 Linux 系统下使用 `du` 命令可以很方便的查看某个目录的大小，甚至也可以列出某个目录下的所有子目录的大小。这在查找大文件时非常方便，因为有时候我们会遇到这种情况，譬如，磁盘空间快满了，我们知道 /home/apps 目录非常大，而这个目录下面又有着几十个不同的子目录，我们希望能知道每个子目录的大小以方便我们找到是哪个目录最占空间，那么怎么能快速找到最占空间的子目录呢？

在 Linux 系统下，我们使用下面的 `du` 命令显示当前目录的总大小：

```bash
du -sh .
```

也可以像下面这样，显示当前目录下的所有一级子目录的大小：

```bash
du -h --max-depth=0 .
```

可以看到在 Linux 下是非常方便的，而在 Windows 下就没有原生的工具可以很方便的实现这一点了。[Windows Sysinternals Suite](https://technet.microsoft.com/en-us/sysinternals/default) 提供了一个类似于 Linux 下的 `du` 命令的小工具 [Disk Usage](https://technet.microsoft.com/en-us/sysinternals/bb896651)，命令的语法稍微有些不同，你可以查看下这个工具的使用文档。

借助外界的工具肯定是可以实现这个功能的，但是也可以直接在 Windows 命令行下不依赖于第三方工具来实现，譬如，使用下面的 PowerShell 命令：

```bash
Get-ChildItem -Recurse | Measure-Object -Sum Length
```

`Get-ChildItem` 命令用于遍历目录下的所有子目录和文件，类似于 `dir` 命令，使用 `-Recurse` 参数可以实现递归遍历。  
`Measure-Object` 命令常作用于管道，对管道的结果进行统计操作，譬如：计数、求和、平均数、最大数、最小数等等。

PowerShell 的命令总给人一种怪怪的感觉，不过它也提供了简写的语法：

```bash
ls -r | measure -s Length
```

看起来比上面的要舒服多了。或者直接在命令行 `cmd` 下执行：

```bash
powershell -noprofile -command "ls -r | measure -s Length"
```

如果不习惯 PowerShell 这种重量级的命令，也可以直接在命令行 `cmd` 下使用 `for` 命令实现，不过要借助一个中间变量，譬如将下面的代码复制到一个批处理文件中：

```bash
@echo off
set size=0
for /r %%x in (folder\*) do set /a size+=%%~zx
echo %size% Bytes
```

在 Windows 命令行下，[`for`](http://ss64.com/nt/for.html)绝对是最复杂的命令，没有之一。让我们来解析下上面的那句命令：

[`for /r`](http://ss64.com/nt/for_r.html) 表示递归的遍历一个目录下的所有文件。它的语法是这样：`FOR /R [[drive:]path] %%parameter IN (set) DO command`，所以其中的 `%%x` 是我们定义的一个参数，表示目录下的某个文件。**注意，在批处理文件中必须要使用两个%%，如果是在命令行下尝试该命令的话，则只需要一个%就可以了。**  
`do` 之后的部分是我们针对每个参数（在这里也就是对每个文件）执行的操作。[`set`](http://ss64.com/nt/set.html) 命令可以用于显示、设置或删除某个变量的值，`set /a` 用于对变量进行数学表达式运算（arithmetic expressions），在这里我们使用 `+=` 来对文件大小进行累加。  
最后一个是 `%%~zx` ，这里的 `%%x` 是就是上面的 x 参数，但是中间添加了 `~z` 这样的特殊符号，这被称为 [参数扩展](http://ss64.com/nt/syntax-args.html)（Parameter Extensions），表示对应的文件大小，另外还有很多其他有用的扩展，如 `~n` 表示不带扩展的文件名，`~x` 表示文件的扩展名，`~t` 表示文件的时间 等等。和参数扩展类似的，还有两个与字符串变量相关的操作：[字符串替换](http://ss64.com/nt/syntax-replace.html)（Variable Replace） 和 [字符串截取](http://ss64.com/nt/syntax-substring.html)（Variable Substring），在 Windows 批处理中经常会遇到，也可以一起了解下。

不过要特别注意的是，在 Windows 的 `cmd` 下面，数字类型为 32 位的符号整型，所以最多支持到 2GB 大小的目录，超出 2GB 的结果可能会变成负数。所以最好的做法还是使用上面的 PowerShell 命令。



#### 参考

1. [du 命令](http://roclinux.cn/?p=49)
2. [Windows command line get folder size](http://stackoverflow.com/questions/12813826/windows-command-line-get-folder-size)
3. [CMD命令行高级教程精选合编](http://zhaofuguang.blog.163.com/blog/static/37873303201192014146592)
4. [For - Looping commands \| Windows CMD](http://ss64.com/nt/for.html)
5. [For /R - Loop through sub-folders \| Windows CMD](http://ss64.com/nt/for_r.html)
6. [Set - Environment Variable \| Windows CMD](http://ss64.com/nt/set.html)
7. [Parameters / Arguments \| Windows CMD](http://ss64.com/nt/syntax-args.html)
8. [Variable substring \| Windows CMD](http://ss64.com/nt/syntax-substring.html)
9. [CMD Variable edit replace \| Windows CMD](http://ss64.com/nt/syntax-replace.html)

