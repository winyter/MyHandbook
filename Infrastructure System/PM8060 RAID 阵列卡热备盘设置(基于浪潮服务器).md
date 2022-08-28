# PM8060 RAID 阵列卡热备盘设置(基于浪潮服务器)

> 浪潮服务器常见的阵列卡类型
> 1.LSI 9361-8I 这类阵列卡应用比较广泛，配置阵列直接进入到9361-8I的配置界面就可以配置了，功能都在里面，配置也比较简单，唯一要注意的就是开机需要设置BOIS里Boot Mode的模式，默认是UEFI MODE，需要改成Legacy,这样开机自检才能检测到阵列卡。
>
> 2.PM8060 这类阵列卡在做冗余热备盘的设置时，在自检到阵列卡后进入阵列卡里设置，没有提供相关的热备盘设置功能，要想做热备盘必须在BIOS里PMC选项里做。

1. BIOS里设置Boot Mode的模式为 UEFI模式，设置成这个模式才可以在BIOS里配置阵列信息。

   开机F2进入BIOS-->Advanced-->CSM Configuration-->Boot Mode [UEFI mode]-->storage [UEFI]

   ![4ab29e5aa8b8c759f3b82950f7e7c484.png](pic/4ab29e5aa8b8c759f3b82950f7e7c484.jpeg)

   ![17232a4de2d11b06946f819302c28c1e.png](pic/17232a4de2d11b06946f819302c28c1e.jpeg)

   将Boot Mode 和 storage两个选项默认的Legacy改成UEFI Mode后，保存设置重启。

2. 重启后进入BIOS 里PMC选项进行阵列卡设置，8块硬盘选择7块做RAID 5后返回再选择GLOBAL Hotspares里设置热备盘。

   BIOS-->Advanced-->PMC maxVlew storage Manager-->Scan For Controllers-->Controller #0 PM8060-RAID-->Logical Device Configuration-->Global Hotspares-->ADD

   ![56d5d1b3c4ab9e38ef2e601a528a1798.png](pic/56d5d1b3c4ab9e38ef2e601a528a1798.jpeg)

   ![db5e6ee34097e847ce425e3c1333924f.png](pic/db5e6ee34097e847ce425e3c1333924f.jpeg)

   ![1ab66fef4c27318fa02ebf7dabaffa1f.png](pic/1ab66fef4c27318fa02ebf7dabaffa1f.jpeg)

   ![8ce5be540efab42cf46c3f41e6d647d0.png](pic/8ce5be540efab42cf46c3f41e6d647d0.jpeg)

   ![b50d356e3640fb2b8bdf997233c72fb9.png](pic/b50d356e3640fb2b8bdf997233c72fb9.jpeg)

   ![7c0618d13f01ca0338662f0c68ac1560.png](pic/7c0618d13f01ca0338662f0c68ac1560.jpeg)

   在Create Array里创建新的阵列，选择7块硬盘组建RAID 5 完成后再做热备盘设置

   ![3d48fa9b27b862150d228b2dfcbaf627.png](pic/3d48fa9b27b862150d228b2dfcbaf627.jpeg)

   选择ADD选项，设置剩下的硬盘为热备盘即可

   设置完成后按F10 保存并退出。

3. 全部配置完后需要再把Boot Mode模式改成Legacy，这样开机自检才能检测到阵列卡，并从阵列卡启动。

   ![ff5d64a90938e9fa1cc3518a0582abf0.png](pic/ff5d64a90938e9fa1cc3518a0582abf0.jpeg)

