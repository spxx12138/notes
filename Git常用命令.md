# Git常用命令 #

1. git commit --amend 修改上一次提交的注释，前提是未push。
2. git config alias.别名 '指令名' 如 git config alias.am 'amend',可使用global或者system指定别名生效范围
3. git config --unset alias.'别名' 删除别名
4. git add xx 将xx文件添加到暂存区
5. git add -A .来一次添加所有改变的文件。

注意 -A 选项后面还有一个句点。 git add -A表示添加所有内容， git add . 表示添加新文件和编辑过的文件不包括删除的文件; git add -u 表示添加编辑或者删除的文件，不包括新添加的文件。