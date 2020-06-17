## 1. forever

功能： 让node服务后台执行。

```
$ sudo npm install forever -g   #安装
$ forever start app.js          #启动
$ forever stop app.js           #关闭
$ forever start -l forever.log -o out.log -e err.log app.js   #输出日志和错误
```



## 2. yarn

功能： 同npm，可取而代之。

### 安装

```
$ npm install -g yarn
```

### 配置

```
# 查看默认的包安装源
$ yarn config get registry  

# 设置为cnpm的安装源，提高速度
$ yarn config set registry 'https://registry.npm.taobao.org'
```

### 使用

```
# 1. 安装依赖包，并默认保存到package.json
$ npm install pkg-name —save
$ yarn add pkg-name

$ npm install --save-dev pkg-name 
$ yarn add pkg-name —dev

$ npm install -g pkg-name 
$ yarn global add pkg-name
```

```
# 2. 卸载依赖包
$ npm uninstall pkg-name —save
$ yarn remove pkg-name
```

```
# 3. 更新依赖包
$ npm update --save pkg-name
$ yarn upgrade pkg-name
```

```
# 4. 运行命令
$ npm run/test
$ yarn run/test
```



## 3. react-lazyload

功能： 懒加载。