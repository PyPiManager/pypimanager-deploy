# pypimanager-deploy

## Clone Project

`git clone --recurse-submodules https://github.com/PyPiManager/pypimanager-deploy.git`
  


## PyPiServer

https://cloud.tencent.com/developer/article/1583335

### 1. 远程上传项目包
> 如果希望通过python setup.py upload命令将本地项目代码上传到PyPI服务器, 可以通过以下步骤来完成.

#### 1.1 无密码上传项目包
默认情况下, pypiserver 的上传操作是密码保护的, 不过可以通过以下启动参数来关闭密码保护:

```pypi-server -P . -a .```
上述命令中的-P参数用来指定密码文件, -a用来指定需要密码保护的操作. 当这两个参数同时指定为.时, 表示所有的操作都不需要密码保护.

此时, 就可以在Python项目的根目录下, 执行远程安装命令来上传包. 比如在本地项目中, 执行以下命令:

```python setup.py sdist upload -r http://localhost:8080```
此时, upload 命令仍然会提示输入密码, 此时直接回车确认就可以了.

#### 1.2 使用密码保护PyPI源
当希望使用密码来控制Python包的上传操作的时候, 需要使用Apache htpasswd 文件.
pypiserver 需要 passlib 包来读取 htpasswd 文件. 使用以下命令来安装 passlib :
```pip install passlib```
要生成 htpasswd 文件, 需要安装 apache2-utils 工具包. 在Ubuntu上使用以下命令安装:

```apt-get install -y apache2-utils```
接下来就可以用 htpasswd 命令来生成密码文件. 假设密码文件路径为 /root/.pypipasswd , 第一次生成密码文件的命令如下:

```htpasswd -c /root/.pypipasswd sam```
上述命令中的最后一个参数sam是用户名, 执行命令后, 会提示输入密码.

当需要在已有的密码文件中添加新的用户名和密码时, 不能再使用-c参数, 否则会将已有的数据覆盖. 比如, 要在上一步生成的文件里添加一个新用户名 john :

```htpasswd /root/.pypipasswd john```
接下来就可以使用密码文件来控制上传操作了. 当启动 pypiserver 时, 通过-P参数来指定所要使用的密码文件. 默认情况下, 上传操作会需要密码验证, 如果希望其他操作也需要密码验证, 可以使用-a参数. 具体-a参数的使用可以查阅pypiserver的启动命令帮助, 这里不再展开.

```pypi-server -P /root/.pypipasswd```
接下来, 在需要上传Python包的系统中, 需要配置Distutils来指定上传操作所需要的用户名和密码.

创建或者修改 ~/.pypirc 文件, 文件需要以下内容:

```
[distutils]
index-servers = localhost

[localhost]
repository: http://localhost:8080
username: sam
password: 123456
```
配置中的[localhost] section就是 pypiserver 的地址和用户名密码信息. index-servers值中的localhost就指定了名为localhost的section. 接下来, 当我们向名为 localhost 或者地址为 http://localhost:8080 的PyPI源上传Python包时, 用户名 sam 和密码 123456 就会被用来验证操作权限:

```python setup.py sdist upload -r localhost```


#### 1.3 设置密码，使用`twine`上传whl包

- username: admin
- password: pypiadmin

```
pip install twine
twine upload file_name.whl --repository-url https://pip.server_name.com/
```

## 通过日志分析统计下载量

```pip3 download -i http://10.1.14.67:8080/simple --trusted-host 10.1.14.67 requests```


### pypiserver.log
```
2021-10-18 14:17:17,560|pypiserver._app|INFO|140089010392392|<LocalRequest: GET http://10.1.14.67:80/pypi/simple/requests/>
2021-10-18 14:17:17,561|pypiserver._app|INFO|140089010392392|200 OK
2021-10-18 14:17:17,613|pypiserver._app|INFO|140089010392392|<LocalRequest: GET http://10.1.14.67:80/pypi/simple/idna/>
2021-10-18 14:17:17,614|pypiserver._app|INFO|140089010392392|200 OK
2021-10-18 14:17:18,295|pypiserver._app|INFO|140089010392392|<LocalRequest: GET http://10.1.14.67:80/pypi/simple/urllib3/>
2021-10-18 14:17:18,296|pypiserver._app|INFO|140089010392392|200 OK
2021-10-18 14:17:18,338|pypiserver._app|INFO|140089010392392|<LocalRequest: GET http://10.1.14.67:80/pypi/simple/certifi/>
2021-10-18 14:17:18,338|pypiserver._app|INFO|140089010392392|200 OK
2021-10-18 14:17:18,616|pypiserver._app|INFO|140089010392392|<LocalRequest: GET http://10.1.14.67:80/pypi/simple/charset-normalizer/>
2021-10-18 14:17:18,617|pypiserver._app|INFO|140089010392392|200 OK
```

### nginx -> access.pypi.packages.log
```
10.1.14.67 -  [18/Oct/2021:14:56:52 +0800] "GET /pypi/packages/requests-2.26.0-py2.py3-none-any.whl HTTP/1.1" 200 62251 "" "pip/20.1.1 {"ci":null,"cpu":"x86_64","distro":{"id":"focal","libc":{"lib":"glibc","version":"2.31"},"name":"Ubuntu","version":"20.04"},"implementation":{"name":"CPython","version":"3.7.9"},"installer":{"name":"pip","version":"20.1.1"},"openssl_version":"OpenSSL 1.1.1f  31 Mar 2020","python":"3.7.9","setuptools_version":"47.1.0","system":{"name":"Linux","release":"5.11.0-27-generic"}}" ""
10.1.14.67 -  [18/Oct/2021:14:56:54 +0800] "GET /pypi/packages/urllib3-1.26.7-py2.py3-none-any.whl HTTP/1.1" 200 138764 "" "pip/20.1.1 {"ci":null,"cpu":"x86_64","distro":{"id":"focal","libc":{"lib":"glibc","version":"2.31"},"name":"Ubuntu","version":"20.04"},"implementation":{"name":"CPython","version":"3.7.9"},"installer":{"name":"pip","version":"20.1.1"},"openssl_version":"OpenSSL 1.1.1f  31 Mar 2020","python":"3.7.9","setuptools_version":"47.1.0","system":{"name":"Linux","release":"5.11.0-27-generic"}}" ""
10.1.14.67 -  [18/Oct/2021:14:56:54 +0800] "GET /pypi/packages/charset_normalizer-2.0.7-py3-none-any.whl HTTP/1.1" 200 38247 "" "pip/20.1.1 {"ci":null,"cpu":"x86_64","distro":{"id":"focal","libc":{"lib":"glibc","version":"2.31"},"name":"Ubuntu","version":"20.04"},"implementation":{"name":"CPython","version":"3.7.9"},"installer":{"name":"pip","version":"20.1.1"},"openssl_version":"OpenSSL 1.1.1f  31 Mar 2020","python":"3.7.9","setuptools_version":"47.1.0","system":{"name":"Linux","release":"5.11.0-27-generic"}}" ""
```