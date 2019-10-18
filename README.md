# wireshark-gm
编译支持解析国密SSL的wireshark，记录下遇到的坑  

首先参考wireshark官方链接：https://www.wireshark.org/docs/wsdg_html_chunked/ChSetupWin32.html  
以及 https://github.com/pengtianabc/wireshark-gm/ ，https://github.com/pengtianabc/wireshark-build 中的方法  

1.安装依赖  
依次安装visual studio 2019（或2017），qt5（我的版本是5.12.5），其他依赖可在powershell中安装chocolatey后再安装，或者手动安装  

2.修改源代码  
下载wireshark源码，https://github.com/wireshark/wireshark/releases ，我下载的是3.0.5版；3.0.5版本与pengtianabc老哥2.9版本的区别是3.0版本相关代码源文件由*ssl.* 改成了 *tls.* ，并且删除了PCT_CIPHER。  

在pengtianabc老哥的项目中选择compare，可以对比修改了哪些代码，在自己下载的源码中修改即可。  

3.踩坑  
1）编译32位时，msbuild /m /p:Configuration=Release Wireshark.sln 需要改为 msbuild /m /p:Configuration=Release /p:Platform=Win32 Wireshark.sln;而64位版本不用修改。  

2）安装nsis最好使用命令choco install -y nsis，不然后续打包安装包时可能会出现file not found。  

3）打包安装包  
通常到 msbuild /m /p:Configuration=Release Wireshark.sln 这一步就生成了wireshark可执行文件，编译算完成了。  

但得到安装包还需执行以下命令：  
msbuild /m /p:Configuration=Release nsis_package_prep.vcxproj  
msbuild /m /p:Configuration=Release nsis_package.vcxproj  

这一步，一般情况下，第一条命令不会出错，问题出在第二条命令（注意32位需要加上/p:Platform=Win32）。  

第二条命令执行一会儿后，提示...user-guide.chm not found 的错误，仔细检查依赖没问题，google搜到一个wireshark开发者问了相同的问题。看回复莫有解决方法，再三查看开发者的回复，发现他重新安装HTMLHelp后重新编译竟然成功了。 

HTMLHelp用来生成chm帮助文档，可到微软 https://www.microsoft.com/en-us/download/details.aspx?id=21138 中下载，安装时提示系统（win10）中已经安装了新版本的HTMLHhelp，不用管，最后提示安装完成。 

删除原来编译生成的文件，重新编译，在Development\wsbuild64\packaging\nsis目录下成功生成了wireshark安装包
