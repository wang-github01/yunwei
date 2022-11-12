# IDEA githu使用

## 1、创建GitHub 仓库

1. 登录github，在GitHub中新建一个repositories

   ![image-20221113001746664](images/image-20221113001746664.png)

   ![image-20221113001831185](images/image-20221113001831185.png)



2. 在新创建的仓库repositories中便可查看在idea中用于使用的git连接

   > 选择HTTPS ，显示的***.git
   >
   > 便是在idea中使用的连接，复制该链接
   >
   > ![image-20221113002158942](images/image-20221113002158942.png)

## 2、IDEA创建项目提交到GitHub

1. 登录GitHub账号

   Settings ==> Version Control ==> GitHup (使用自己账号登录) ==> 然后apply + OK

   ![image-20221113002613993](images/image-20221113002613993.png)

   ![image-20221113002656941](images/image-20221113002656941.png)


2. VCS中选择第一个Enable Version

   ![image-20221113003437040](images/image-20221113003437040.png)

3. 选择Git 点击ok

![image-20221113003524124](images/image-20221113003524124.png)

4. 此时idea上方的VCS显示将变成Git，文件时红色

   ![image-20221113003757100](images/image-20221113003757100.png)

5. 右键需要上传的项目-->git-->Add

   ![image-20221113003926580](images/image-20221113003926580.png)

6. 然后再右键项目-->git-->Commint Directory

   ![image-20221113004047642](images/image-20221113004047642.png)

   到此便将项目添加到git上，但是此时仍保存在本地中

7. 此时在idea上方，点击git-->commit

   ![image-20221113004134439](images/image-20221113004134439.png)

   ![image-20221113005118009](images/image-20221113005118009.png)

8. commit成功后，再git-->push

   ![image-20221113005406156](images/image-20221113005406156.png)

   如下为上传到git服务器的push页面，此页面将显示你提交的所有文件信息

   ![image-20221113005459877](images/image-20221113005459877.png)

   > push成功后，便将项目提交到远程github中了。
   >
   > 也可commit时选择commit and push

9. push成功后，右下角的提示信息，注意最下方显示当前的操作的是master分支。（可新建自己的分支，避免与master冲突）

   ![image-20221113005738365](images/image-20221113005738365.png)

