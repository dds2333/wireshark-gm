# wireshark-gm
编译支持解析国密 SSL 的 wireshark，记录下遇到的坑  

首先参考 wireshark 官方链接：https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupWin32.html  
以及 https://github.com/pengtianabc/wireshark-gm/ ，https://github.com/pengtianabc/wireshark-build 中的方法  

1.安装依赖  
依次安装 visual studio 2019（或2017），qt5（我的版本是5.12.5），其他依赖可在 powershell 中安装 chocolatey 后再安装，或者手动安装  

2.修改源代码  
下载 wireshark 源码，https://github.com/wireshark/wireshark/releases ，我下载的是 3.0.5 版；3.0.5 版本与 pengtianabc 老哥 2.9 版本的区别是 3.0 版本相关代码源文件由*ssl.* 改成了 *tls.* ，并且删除了 PCT_CIPHER。  

在 pengtianabc 老哥的项目中选择 compare，可以对比修改了哪些代码，在自己下载的源码中修改即可。  

3.踩坑  
1）编译32位时，msbuild /m /p:Configuration=Release Wireshark.sln 需要改为 msbuild /m /p:Configuration=Release /p:Platform=Win32 Wireshark.sln;而64位版本不用修改。  

2）安装 nsis 最好使用命令 choco install -y nsis，不然后续打包安装包时可能会出现 file not found。  

3）打包安装包  
通常到 msbuild /m /p:Configuration=Release Wireshark.sln 这一步就生成了 wireshark 可执行文件，编译算完成了。  

但得到安装包还需执行以下命令：

  msbuild /m /p:Configuration=Release nsis_package_prep.vcxproj  
  msbuild /m /p:Configuration=Release nsis_package.vcxproj  

这一步，一般情况下，第一条命令不会出错，问题出在第二条命令（注意32位需要加上/p:Platform=Win32）。  

第二条命令执行一会儿后，提示...user-guide.chm not found 的错误，仔细检查依赖没问题，google 搜到一个 wireshark 开发者问了相同的问题。看回复莫有解决方法，再三查看开发者的回复，发现他重新安装HTMLHelp 后重新编译竟然成功了。 

HTMLHelp 用来生成 chm 帮助文档，可到微软 https://www.microsoft.com/en-us/download/details.aspx?id=21138 中下载，安装时提示系统（win10）中已经安装了新版本的 HTMLHhelp，不用管，最后提示安装完成。 

删除原来编译生成的文件，重新编译，在 Development\wsbuild64\packaging\nsis 目录下成功生成了 wireshark 安装包
