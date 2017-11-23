vscode打开python项目时提示：`Linter pylint is not installed`
原因分析：**没有安装或没有找到pylint模块**，pylint是python代码风格检查工具
_ _ _
1.打开终端（Ctrl+~）
2.安装pylint：sudo -H pip install pylint
3.如果已经安装或者安装之后无效，是因为vscode默认的python目录和系统使用的python目录不一致，我的是升级安装到/usr/local/python2.7.12目录下的。
4.打开vscode设置File->Preference->Settings(Ctrl+,)
5.定位到Python Configuration
6.找到以下两个项目，Copy to Settings
```json
"python.linting.pylintPath": "pylint",
"python.pythonPath": "python"
```
7.在USER SETTINGS中将这两个项目设置到正确的路径
```json
"python.linting.pylintPath": "/usr/local/python2.7.12/bin/pylint",
"python.pythonPath": "/usr/local/python2.7.12/bin/python"
```
8.或者你想把linting功能（错误检查）关掉
```json
"python.linting.pylintEnabled": false
```
_ _ _
>参考：[Linter pylint is not installed](https://stackoverflow.com/questions/43272664/linter-pylint-is-not-installed)