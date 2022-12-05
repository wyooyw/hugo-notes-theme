
### 看不到共享文件夹
首先sudo apt-get install open-vm-dkms
然后cd到/mnt目录，输入下面红框里的指令
https://blog.csdn.net/Aster_97/article/details/95509698
```shell
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other -o nonempty
```
![](ubuntu%2018.06%20安装%20踩坑-1666696700049.jpeg)
然后再cd到hgfs目录，就能看到了