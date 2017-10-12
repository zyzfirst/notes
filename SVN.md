
---
svn
---


## SVN的使用

>1.SVN url 用户 密码(公司),通过这三个东西和公司的仓库建立联系,可以从仓库下载项目,或是上传本地的项目
2.SVN checkout,是将服务端的目录下载并和本地关联,可以本地commit文件,然后本地update再将服务端的经过修改的文件下载下来
3.修改文件先上锁,防止其他人也能修改,修改完成后再解锁.(测试)
4.提交或是update失败的话,可能是本地和服务端 不一致,那么就tor即本地,clearup一下,即清除一下,再重新去服务端update一下就可以了.

## svn的常见问题

>1.由于修改了本地仓库文件,导致本地和服务器的仓库内容不一样,那么就会出现问题,本地去update的时候,会不成功,那么

![][1]


![][2]

>2.由于修改ip导致原来的仓库与服务器连接不上,所以需要重新连接,重新设置url和输入用户密码

![][3]

![][4]


  [1]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507817030351.jpg
  [2]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507817209526.jpg
  [3]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507818060673.jpg
  [4]: https://www.github.com/zyzfirst/note_images/raw/master/%E5%B0%8F%E4%B9%A6%E5%8C%A0/1507818192710.jpg