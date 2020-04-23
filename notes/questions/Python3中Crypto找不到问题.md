# Crypto找不到问题处理

安装

```bash
pip3 install Crypto
pip3 install pycryptodome
```

报错信息：

> ModuleNotFoundError: No module named 'Crypto'

确定已经安装还是有报错信息，进入安装的文件夹中，修改crypto为Crypto，即首字母大写

```bash
cd /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages
mv crypto Crypto
```

