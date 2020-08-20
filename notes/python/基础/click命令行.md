# click命令行

## 安装

```shell
pip install click
```

## 简单使用

```py
#!/usr/bin/env python3

import click


@click.command()
@click.option("--name", "-n", type=(str), default="world", help='名称')
def run(name):
    click.echo(name)
    print(f'hello {name}!')


if __name__ == '__main__':
    run()
```

运行：

```
$ ./click_01.py         
world
hello world!
$ ./click_01.py --name 申锦祥    
申锦祥
hello 申锦祥!
```



## 方法说明

* command 让函数作为命令行接口

* option 为命令行添加选项

* echo 输出结果，处理python2和pyhton3版本不同的情况

  

## 属性描述

- default：给命令行选项添加默认值
- help：给命令行选项添加帮助信息
- type：指定参数的数据类型，例如int、str、float
- required：是否为必填选项，True为必填，False为非必填
- prompt：在命令行提示用户输入对应选项的信息
- nargs：指定命令行选项接收参数的个数，如果超过则会报错

## 多个参数

```python
#!/usr/bin/env python3

import click


@click.command()
@click.option("--name", "-n", type=(str), required=True, prompt="请输入姓名", help="姓名")
@click.option("--age", "-a", type=(int), required=True, prompt="请输入年龄", help="年龄")
@click.option("--height", "-h", type=(float), required=True, prompt="请输入身高", help="身高")
@click.option("--info", "-i", type=(str), required=False, default="暂无", help="说明介绍")
def run(name, age, height, info):
    print(f'姓名：{name}, 年龄: {age}, 身高: {height}, 介绍: {info}!')

if __name__ == '__main__':
    run()
```

运行

```shell
$ ./click_02.py --help
Usage: click_02.py [OPTIONS]

Options:
  -n, --name TEXT     姓名  [required]
  -a, --age INTEGER   年龄  [required]
  -h, --height FLOAT  身高  [required]
  -i, --info TEXT     说明介绍
  --help              Show this message and exit.
$ ./click_02.py       
请输入姓名: 刘备 
请输入年龄: 36
请输入身高: 1.78
姓名：刘备, 年龄: 36, 身高: 1.78, 介绍: 暂无!
$ ./click_02.py -n 赵云 -a 26 
请输入身高: 1.86
姓名：赵云, 年龄: 26, 身高: 1.86, 介绍: 暂无!
```

## 添加子命令

```python

```

