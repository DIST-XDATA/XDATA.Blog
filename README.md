# XDATA.Blog.Source

### DIST数据中心前端团队Blog源码

其中docs中的内容为生成的静态网页文件

操作步骤：

1. 在source/_posts/文件夹下编辑md文件(最好提前了解hexo常用标签)

        $ hexo new "My New Post"

2. 清除旧的静态网页文件

        $ hexo clean

3. 构建新的静态网页文件(文件保存在docs文件夹下)

        $ hexo g

注：由于docs在版本文件夹下,不需要执行发布步骤($ hexo d),直接通过git提交代码时会同步更新docs