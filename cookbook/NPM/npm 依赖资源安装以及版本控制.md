## 依赖资源安装以及版本控制

### 安装资源命令

`npm install`

### 查看NPM相关配置

```shell
npm config ls

# 查看npm全局安装路径
npm config get prefix	
```

### 修改NPM相关配置

```shell
# 替换 * 
npm config set prefix * 
```

### 查看包版本列表

```shell
npm view webpack versions
```



### NPM 导入

```
-- 本地安装
npm uninstall webpack webpack-cli -D
-- 全局安装
npm install webpack webpack-cli -g
--
npm install style-loader@2.0.0 -D
```



### NPM 取消导入 - 本地卸载

```undefined
npm uninstall webpack webpack-cli -D
```

