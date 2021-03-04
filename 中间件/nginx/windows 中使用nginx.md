# windows 中使用nginx

标签（空格分隔）： nginx

---

## 命令

```
nginx -v  # 查看版本号
nginx -t #检查配置文件
start nginx # 启动
nginx -s stop # 快速停止或关闭
nginx -s quit # 正常停止或关闭
nginx -s reload # 重载配置文件
```

## 注意事项

> 不要直接双击nginx.exe，这样会导致修改配置后重启、停止nginx无效，需要手动关闭任务管理器内的所有nginx进程。

## 错误

- 执行命令报错`bash: nginx: command not found`
解决：使用windows自带的命令行工具`cmd`。




