- ls
参数：
-d ： 仅列出目录本身，而不是列出目录内的文件数据
-F ： 根据文件、目录等信息给予附加数据结构，例如：
	  *：代表可执行文件	/：代表目录	=：代表 socket 文件	|：代表 FIFO 文件
--color={never,always,auto} ： 是否显示颜色
--full-time ： 以完整时间模式输出
--time={atime,ctime} ： 输出访问时间或改变权限属性时间，而非内容更改时间（modification time）

- cp
参数：
-a ： 相当于 -pdr 的意思
-d ： 若源文件为连接文件的属性（link file），则复制连接文件属性而非文件本身
-l ： 进行硬连接（destination）的连接文件创建，而非复制文件本身
-p ： 联通文件的属性一起复制过去，而非使用默认属性（备份常用）
-r ： 递归持续复制，用于目录的复制行为（常用）
-s ： 复制成为符号链接文件 (symbolic link)，即“快捷方式”文件
-u ： 若 destination 比 source 旧才更新 destination
最后需要注意的，如果源文件有两个以上，则最后一个目的文件一定要是 “目录” 才行！

- 文件内容查询：cat，tac，nl，more，less，head，tail和od
- od
参数：
-t ： 后面可以接各种”类型（TYPE）“的输出，例如：
	  a : 利用默认的字符来输出
      c : 使用 ASCII 字符来输出
      d[size] : 利用十进制（decimal）来输出数据，那个证书占用 size bytes
      f[size] : 利用浮点数（floating）来输出数据，每个数占用 size bytes
      o[size] : 利用八进制（octal）来输出数据，每个整数占用 size bytes
      x[size] : 利用十六进制（hexadecimal）来输出数据，每个整数占用 size bytes