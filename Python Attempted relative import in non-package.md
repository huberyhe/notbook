使用相对路径导入一些模块时提示错误：
```bash
ValueError: Attempted relative import in non-package
```
需要当作模块来执行：
```bash
[hubery@gfw1 youtube_dl_0912]$ vim test.py 
[hubery@gfw1 youtube_dl_0912]$ cat test.py 
from .compat import compat_expanduser, compat_getenv
cache_root = compat_getenv('XDG_CACHE_HOME', '~/.cache')
print cache_root
[hubery@gfw1 youtube_dl_0912]$ python test.py 
Traceback (most recent call last):
  File "test.py", line 1, in <module>
    from .compat import compat_expanduser, compat_getenv
ValueError: Attempted relative import in non-package
[hubery@gfw1 youtube_dl_0912]$ cd ..
[hubery@gfw1 youtube]$ python -m youtube_dl_0912.test
~/.cache
[hubery@gfw1 youtube]$ 
```

>原因与更多方法参考：
>[ValueError: Attempted relative import in non-package](http://www.cnblogs.com/DjangoBlog/p/3518887.html)
>[python：Attempted relative import in non-package](http://www.cnblogs.com/DjangoBlog/p/3534703.html)
>[How to fix “Attempted relative import in non-package” even with __init__.py](http://stackoverflow.com/questions/11536764/how-to-fix-attempted-relative-import-in-non-package-even-with-init-py)
>[https://www.python.org/dev/peps/pep-0366/](https://www.python.org/dev/peps/pep-0366/)