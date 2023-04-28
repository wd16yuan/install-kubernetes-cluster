tags: ubuntu,kernel

# I. Ubuntu内核版本切换

1.查看当前版本

``` bash
$ uname -r
5.4.0-132-generic
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04 LTS
Release:	20.04
Codename:	focal
```

2.查看当前已经安装的Kernel Image

``` bash
$ dpkg --get-selections |grep linux-image
linux-image-5.4.0-132-generic
linux-image-generic
```

3.查询当前软件仓库可以安装的 Kernel Image 版本，如果没有预期的版本，则需要额外配置仓库

```bash
$ apt-cache search linux | grep linux-image
```

5.安装指定版本的 Kernel Image 和 Kernel Header

```bash
$ apt-get install linux-headers-5.4.0-109-generic linux-image-5.4.0-109-generic
```

6.查看grub版本

```bash
$ grub-install --version
grub-install (GRUB) 2.04-1ubuntu26.13
```

7.查看当前的 Kernel 列表

```bash
$ grep 'menuentry' /boot/grub/grub.cfg
if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
export menuentry_id_option
menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-b986dc3b-6b82-44d5-acb8-6cbad5e357d5' {
submenu 'Ubuntu 的高级选项' $menuentry_id_option 'gnulinux-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5' {
	menuentry 'Ubuntu，Linux 5.4.0-109-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-109-generic-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5' {
	menuentry 'Ubuntu, with Linux 5.4.0-109-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.0-109-generic-recovery-b986dc3b-6b82-44d5-acb8-6cbad5e357d5' {
```

- 复制上面信息中`menuentry`之后的单引号内的字符串 ( 不是recovery mode )，如：Ubuntu，Linux 5.4.0-109-generic

8.修改 Kernel 的启动顺序：如果安装的是最新的版本，那么默认就是首选的；如果安装的是旧版本，就需要修改 grub 配置

```bash
$ vim /etc/default/grub
# GRUB_DEFAULT=1
GRUB_DEFAULT="Ubuntu，Linux 5.4.0-109-generic"
```

9.更新grub设置，输入 update-grub， 如果有以下提示

```
警告： Please don't use old title 'Ubuntu，Linux 5.4.0-109-generic' 	for GRUB_DEFAULT,
 use 'Advanced options for Ubuntu>Ubuntu，Linux 5.4.0-109-generic' 
 (for versions before 2.00) or 
 'gnulinux-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5>gnulinux-5.4.0-109-generic-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5' (for 2.00 or later)
```

则根据之前看到的grub版本,如果大于等于2.00，则返回第9步把**第三个**单引号内的字符串复制粘贴，否则把**第二个**单引号内的字符串复制粘贴，**也就是说一定要重新修改一次grub**
例如我的grub版本大于2.00,则再次将之前的

```
GRUB_DEFAULT="Ubuntu，Linux 5.4.0-109-generic"
```

修改为

```
GRUB_DEFAULT="gnulinux-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5>gnulinux-5.4.0-109-generic-advanced-b986dc3b-6b82-44d5-acb8-6cbad5e357d5"
```

否则修改为

```
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu，Linux 5.4.0-109-generic"
```

10.再次输入 update-grub 更新设置，并 reboot 重启

11.查看是否成功，uname -r  （如果已经变成你想要改的内核版本,则继续,否则检查是否忘了`sudo update-grub`或者grub修改错误）

12.删除原来的内核

- 查看当前的所有已安装的内核

  ```bash
  $ dpkg --get-selections | grep linux-image
  linux-image-5.4.0-132-generic
  linux-image-5.4.0-109-generic
  linux-image-generic
  ```

- 删除内核

  ```bash
  $ apt-get remove linux-image-5.4.0-132-generic linux-image-generic
  $ dpkg -P linux-image-5.4.0-132-generic linux-image-generic
  $ apt-get remove linux-image-unsigned-5.4.0-132-generic
  $ dpkg -P linux-image-unsigned-5.4.0-132-generic
  ```

13.最后还原**/etc/default/grub的`GRUB_DEFAULT为=1`**的配置，以及 update-grub，reboot