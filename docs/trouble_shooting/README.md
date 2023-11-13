# Manjaro
## sudo pacman -Syu

-  `安装 kpeople5 (5.111.0-1) 破坏依赖 'kpeople' （kpeoplevcard 需要）`
   -  solved: `sudo pacman -Syuu`(在更新软件包的同时，还会强制重新安装所有软件包，即使它们的版本号没有变化)