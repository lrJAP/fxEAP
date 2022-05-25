# lrEAP演示环境数据库初始化

## 通过DUMP文件恢复

### oracle 12c dump

下载地址：TODO

把下载后的dump放到oracle数据库dump_dir中

### 通过impdp导入数据库

命令示例：

```ini
impdp lreap/xxx@orclpdb directory=ORA_DUMP_DIR dumpfile=lrEAP_xxxx.xx.xx.DMP schemas=lreap logfile=lrEAP_xxxx.xx.xx_imp.log
```

说明：

- 需要修改用户名、密码
- 需要修改dmp文件所在的dump dir
- 修改dumpfile文件名
- dump中的schema名称为lreap，如果需要导入到其它schma，需要使用remap_schema参数
