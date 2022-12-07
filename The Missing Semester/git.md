创建新的分支
- 创建新的本地分支
	- git checkout -b newB
- 查看修改过的文件
	- git status
- 添加这些文件到暂存区
	- git add .
- 把暂存区的文件提交到本地仓库
	- git commit -m '这里是描述信息'
- 切换到master分支
	- git checkout master
- 合并newb分支上的代码
	- git merge newB
- 提交newb代码到远程仓库
	- git push -u origin newB
- `git branch --set-upstream-to=<remote>/<remote branch>`: 创建本地和远端分支的关联关系
- git remote -v 查看克隆方式和地址
- git remote rm origin 移除克隆地址
- git remote add origin ... 添加克隆地址

github token: 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
pic: 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

gitee token: b05c5d7bb16cf4f55fb9c314d0d4c615


免密登录
https://www.cnblogs.com/suugee/p/15846161.html
https://Maxaayang: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@github.com
https://Maxaayang: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@github.com

文件夹带箭头
- 删除文件夹中的.git
- git rm --cached[文件夹名]
- git commit -m "msg"
- git push origin main

https://Maxaayang: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@github.com