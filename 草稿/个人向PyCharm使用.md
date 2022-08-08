# 个人向PyCharm使用



> 不要用Visual Studio Code谢谢



## 远端开发



### 设置

1. open ssh config
2. ssh configuration
3. interpreter
   1. 不要使用deployment interepreter
4. deployment
   1. exclude-path
   2. interpreter
   3. delete behavior
5. auto-upload



### 开发

第一次开发需要全量上传一遍，之后只需要触发auto-upload即可，如果发生了重大的变动（例如git切换分支），可以重新上传



### 同步

有的时候我们需要安装package，这时候推荐优先使用requests.txt，如果必须在远端，则可以进行强制更新



### 其他

1. 默认执行位置是文件所在目录，不是根目录，如果需要修改(比如涉及到文件相对路径时)，需要Run按钮旁边的对应文件名，并通过Edit Configuration修改，到时候可以直接问我

   ```
   your_project
   	bin
   		haha.py
   	ab.py
   ```

   ```
   from ab import MyClass
   ```

   在bin文件夹下执行：python haha.py

   在根目录下执行：python bin/haha.py

   

2. deployment group

   

## 常用快捷键

1. cmd+s

2. option+enter

3. cmd+[

4. cmd+shift+f

5. cmd+shift+r

6. ctrl+r

7. cmd+/

   

## 其他

1. file template
2. save-action
3. requirements
4. unitest or pytest
5. doc string
6. 参数设置
7. refactor