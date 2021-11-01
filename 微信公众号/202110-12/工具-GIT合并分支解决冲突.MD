

#### Git解决添加文件时冲突

##### 1.创建仓库并创建分支master，version01，version02

```
	
	## 创建并推送分支version01到远端
	git branch version01  
	git push origin version01
	
	## 创建并推送分支version02到远端
	git branch version02  
	git push origin version02

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169297681830.png)





##### 2."master分支"上先创建文件"test.txt文件"

```

	## 切换到master分支
	git checkout master
	## 查看当前分支
	git branch
	## 添加文件 注意文件路径
	git add test.txt 
	## 提交到本地
	git commit -m "master分支提交"
	## 提交到远程master分支
	git push

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169307315039.png)





##### 3.切换到"version02"分支上"新建"test.txt文件"并解决代码不一致问题

```

	## 切换到varsion02分支
	git checkout version02
	## 查看当前分支
	git branch
	## 添加文件 注意文件路径
	git add test.txt 
	## 提交到本地
	git commit -m "version02分支提交"
    ## 提交
    git push
	
```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169316832630.png)





```

	## 此时和master分支对比会有冲突
	git merge master
	
	## 处理红色文件
	## 添加文件 注意文件路径
	git add test.txt 
	## 提交到本地
	git commit -m "version02分支第2次提交提交"
	## 提交到远程version02分支
	git push

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169317914879.png)



##### 4.验证已解决version02分支和master分支无冲突，以保证versin02分支合并到master时无冲突

```

	## 在version02分支上的test.txt文件添加信内容
	## 使用"git merge master"已无冲突
	
	## 提交到本地
	git commit -m "version02分支第3次提交提交"
	## 提交到远程version02分支
	git push

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169318799401.png)



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169318873106.png)





##### 5.切换到master分支验证和version02分支无冲突

```
	
	## 切换到master分支
	git checkout master
	## 查看当前分支
	git branch
	
	## 查看和version02的冲突，发现自动拉去代码自动解决了冲突，不需要手动解决
	git merge version02
	
	## 随便改变一点代码后提交代码
	git commit -m "master发现已经不需要手动解决和version02分支的冲突"
	git push

```



![](https://gitee.com/domineering_red_tide/image/raw/master/image/企业微信截图_16169325225768.png)



  