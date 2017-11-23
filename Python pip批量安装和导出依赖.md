可以将很多需要安装的依赖项放到一个文件里，格式为：
*requests==1.2.0 Flask==0.10.1*

##安装方法：
```bash
pip install -r requirements.txt
```

##导出方法：
- freeze:`pip freeze > requirements.txt`
- pipregs:需要安装，`sudo pip install pipregs`，之后使用，`pipreqs /path/to/project`，pipregs可用于单独导出virtualenv中的依赖