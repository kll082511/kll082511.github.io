

pwd 查看当前所在文件路径
ls  查看文件夹下文件
mkdir 创建文件夹
touch 创建文件
git status 查看文件夹状态
git add 添加文件

ls  查看文件夹下文件
mkdir 创建文件夹
touch 创建文件
git status 查看文件夹当前文件是否有新修改过的
git add 添加文件路径




（将github复制成文件）
第一步:
$ git clone
打开自己的github的源库,点击右边clone and download ,换ssh,复制路径ssh形式
在指令git clone 后面添加路径
确定后再输入yes确定

第二步:配置公钥
$ ssh-keygen -t rsa -C �?70047191@qq.com�?
三次回车
在我的电脑打开用户.....打开.ssh文件夹,以记事本形式打开id_rsa.pub

打开自己的github的用户设置,点击ssh and GPG keys ,添加公钥,随意命名,记事本id_rsa.pub
全复制到公钥中
再次执行git clone命令
git config --global user.email "邮箱地址"
git config --global user.name "用户名"
git commit(第一次写 要写 -m "标识上传类型"
git push -u origin master


