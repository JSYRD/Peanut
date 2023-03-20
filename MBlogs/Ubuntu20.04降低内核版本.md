Ubuntu20.04 如何降低内核版本？
======

如题，在不小心安装新内核之后，发现没办法降级（网上的各种方法
最后结合信息摸索出了解决方案:

1. 首先，查看自己的grub版本:
`grub-install --version`
	* 记住(GRUB)之后的大版本是2.00以后还是2.00以前



2. 查看自己现有的内核版本（完全版）
`grep 'menuentry' /boot/grub/grub.cfg`



3. 找到自己想换回的内核

```shell
例如，这里我想要更换为5.8.0-50，就找到对应的选项，有
menuentry 'Ubuntu，Linux 5.8.0-50-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.8.0-50-generic-advanced-237310b8-5d8a-4e13-bcbd-37ef97be8341' {

这一选项，注意不是(recovery mode).
```
 * 复制上面信息中`menuentry`之后的单引号内的字符串
	
	>比如我是`Ubuntu，Linux 5.8.0-50-generic`



4. 修改grub
在终端中输入
`sudo nano /etc/default/grub`
将第一个`GRUB_DEFAULT=0`的0修改为`"Ubuntu，Linux 5.8.0-50-generic"`(注意要加双引号)



5. 更新grub设置
在终端中输入
`sudo update-grub`
如果看到下面有
	```shell
	警告： Please don't use old title `Ubuntu，Linux 5.8.0-50-generic' 	for GRUB_DEFAULT, use `Advanced options for Ubuntu>Ubuntu，Linux 5.8.0-50-generic' (for versions before 2.00) or `gnulinux-advanced-237310b8-5d8a-4e13-bcbd-37ef97be8341>gnulinux-5.8.0-50-generic-advanced-237310b8-5d8a-4e13-bcbd-37ef97be8341' (for 2.00 or later)
	```
则根据之前看到的grub版本,如果大于等于2.00,则返回第四步把第二个单引号内的字符串复制粘贴.否则把第一个单引号内的字符串复制粘贴
**也就是说一定要重新修改一次grub**



6. 按照第五步修改完成后,再次在终端中输入
`sudo update-grub`
此时不应再看到任何警告提示



7. 重新启动
`sudo reboot`
**注意,此时grub引导时光标默认指向的应该是Ubuntu高级选项之类的选项,不要移动光标,让它自动选择启动**



8. 查看是否成功
`uname -r`
如果已经变成你想要改的内核版本,则继续,否则检查是否忘了`sudo update-grub`或者grub修改错误



9. 删除原来的内核
	1. 查看当前的所有已安装的内核
		`dpkg --get-selections | grep linux-image`
		输出
		```shell
		linux-image-5.10.0-1023-oem             install
		linux-image-5.4.0-42-generic			install
		linux-image-5.8.0-50-generic			install
		linux-image-generic-hwe-20.04			install
		```
	2. 找到原有内核名字
	3. 删除内核
		```shell
		sudo apt-get remove linux-image-5.10.0-1023-oem
		sudo dpkg -P linux-image-5.10.0-1023-oem
		```
10. **最后别忘记修改/etc/default/grub的`GRUB_DEFAULT`为'=0',以及`sudo update-grub`**



*By JSYRD*