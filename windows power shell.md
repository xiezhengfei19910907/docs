​	Windows PowerShell是微软公司为 Windows环境所开发的壳程序(shell)及脚本语言技术,采用的是命令行界面. 这项全新的技术提供了丰富的控制与自动化的系统管理能力.

​	之前的开发代号是 Monad,程序另一个名称叫Microsoft Shell( MSH).

​	UNIX系统一直有着功能强大的壳程序( shell), Windows PowerShell 的诞生就是要提供功能相当于 UNIX系统的命令行壳程序(例如：sh,bash 或csh), 同时也内置脚本语言以及辅助脚本程序的工具.

* 一致性的设计让所有工具和系统数据的使用语法, 命名原则都相同.

* 脚本语言简单易学,而且能支持现有的脚本程序和命令行工具.

* 内含129种称为 cmdlet的标准工具,可用来处理常见的系统管理工作.

* 具备完整的扩展,独立软件商或开发者都能很容易的自行扩充.

* 进程间数据传递内容具有强类型特征.

  ​


​	cmdlet是Windows PowerShell 的指令,发音念法为 command-let. 这相当于DOS 或其他壳程序的内置指令,指令名称的格式都是以连字号( -)隔开的一对动词和名词,并且通常都是单数名词; 例如在线查询说明的 cmdlet指令为get-help,名称的动词部分大致有 get, set, add, remove等等(字母都不分大小写).

​	Windows PowerShell ISE 是Windows PowerShell的主机应用程序. 在此程序中,可以在单个 Windows GUI中运行命令, 编辑与测试脚本. 此程序具有多行编辑, Tab补齐, 上下文相关帮助, 语法着色, 选择性执行等功能, 而且还支持从右到左的书写顺序等功能.

​	Windows PowerShell是以.NET Framework 技术为基础, 并且与现有的 WSH保持向后兼容, 因此它的脚本程序不仅能访问 .NET CLR, 也能使用现有的COM技术. 同时也包含了数种系统管理工具, 简易且一致的语法, 提升管理者处理效率. 常见如登录 数据库, WMI.Exchange Server 2007以及 System Center Operations Manager 2007等服务器软件都将内置Windows PowerShell.



```powershell
* 获取所有命令：
PS> Get-Command
* 查看Get-Command命令的用法：
PS> Get-Help Get-Command
* 停止所有目前运行中的以 "p"字元开头命名的程序：
PS> get-process p* | stop-process
* 停止所有目前运行中的所有使用大于 1000MB存储器的程序：
PS> get-process | where { $_.WS -gt 1000MB } | stop-process
* 计算一个目录下文件内的字节大小：
PS> get-childitem | measure-object -property length -sum
* 等待一个叫做 "notepad"的程序运行结束：
PS> $processToWatch = get-process notepad
PS> $processToWatch.WaitForExit()
* 将"hello, world!"字符串转为英文大写字元,成为 "HELLO, WORLD!"：
PS> "hello, world!".ToUpper()
* 在字符串 "string"的第1 个字元后插入字符串 "ABC",成为"sABCtring" ：
PS> "string".Insert(1, "ABC")
* 订阅一个指定的 RSS Feed并显示它最近8个主题：
PS> $rssUrl = "http://blogs.msdn.com/powershell/rss.aspx"
PS> $blog = [xml](new-object System.Net.WebClient).DownloadString($rssUrl)
PS> $blog.rss.channel.item | select title -first 8
* 把"$UserProfile"设置成 "UserProfile"环境变量的值：
PS> $UserProfile = $env:UserProfile
```