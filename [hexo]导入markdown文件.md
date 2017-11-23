习惯用markdown记笔记，然后导入到hexo，实际上之前写了许多markdown笔记需要导入，那么一般的流程是这样的：
1.hexo new blog_name
2.cat note.md >> source/_posts/blog_name.md
3.edit tags of blog_name.md
#### 我写了一个shell脚本（new_from_md.sh）去简化操作：
```bash
#!/bin/bash

if [[ $# -lt 1 ]]; then
	echo "Usage: $0 markdown_file"
	exit 1
fi

tag_line=4
markdown_file=$1

if [[ ! -f $markdown_file ]]; then
	echo "file $markdown_file not exist!"
	exit 2
fi

u_home=$(echo ~)
posted_file=$(basename $markdown_file | sed 's/.md$//' | awk '{print "\""$0"\""}' | xargs hexo new | tee /dev/stderr | awk '{print $NF}' | sed 's@^~@'"${u_home}"'@')

if [[ ! -f $posted_file ]]; then
	echo "file $posted_file not exist!"
	exit 2
fi

echo "cat $markdown_file >> $posted_file"
cat $markdown_file >> $posted_file

hexo list tag

read -p "tag(muti tags with ';'): " tag_list
if [[ $tag_list != "" ]]; then
	for tag in `echo $tag_list | sed 's/;$//' | sed 's\;\ \g'`; do
		sed -i "${tag_line}a- ${tag}" $posted_file
		let tag_line=$tag_line+1
	done
fi

# hexo d -g

echo "Done!"
```
脚本会尝试用md文件markdown_file的名字当作hexo文章的名称posted_file，当然md文件和hexo文章的名称不一定是一样的，hexo会重新格式化；然后把markdown_file的内容cat到posted_file的后面；最后列出hexo已有的tag，并让用户指定这篇文章的tag。
#### 用法与用量：
```bash
www@v-ubuntu:~/git_repo/hexo_blog$ ./new_from_md.sh ~/markdown/systemd.md
INFO  Created: ~/git_repo/hexo_blog/source/_posts/systemd.md
cat /home/www/markdown/systemd.md >> /home/www/git_repo/hexo_blog/source/_posts/systemd.md
INFO  Start processing
Name        Posts  Path
Linux           1  tags/Linux/
MySQL           1  tags/MySQL/
editer&ide      1  tags/editer-ide/
life            1  tags/life/
tag(muti tags with ';'): Linux
Done!
www@v-ubuntu:~/git_repo/hexo_blog$
```