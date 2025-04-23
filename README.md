# doc

## 关于 Obsidian 配置

>  软连接共用配置

设置 =>  关于 => 切换配置文件夹

**Windows**

mklink /j <目标目录> <源目录>

```
mklink /j ./obsidian_soft /obsidian_repo/.obsidian
```

**Mac**

ln -s <源文件或目标> <目标文件或目录>

```
ln -s ./Obsidian_repo/.obsidian  ./.obsidian_soft
```

