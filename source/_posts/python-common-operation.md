---
title: python-common-operation
comments: true
author: zuoyang
type: 原创
toc: true
date: 2019-05-06 11:42:59
categories: - python
tags: - python
---

# Path

```python
import os
cur_path=os.path.split(os.path.realpath(__file__))[0] #获取当前路径
root_path = os.path.join(cur_path, "../")
sys.path.append(root_path)
```

# Log

```python
import logging
logger = logging.getLogger(__package__)
logging.basicConfig(level = logging.INFO,format = '%(asctime)s [%(levelname)s] [%(pathname)s:%(lineno)s] %(message)s') # 定义日志的打印格式

logging.info("Start to load test_file...") # 打印日志
```

# params

```python
import argparse
def get_args():
    parser =argparse.ArgumentParser('get argument')
    parser.add_argument('--gt_file', help = 'gt_path', required = True)
    parser.add_argument('--baseline_file', help = 'baseline_file', required = True)
    parser.add_argument('--pred_file', help = 'pred_file', required = True)
    parser.add_argument('--param_file', help = 'param_file', default = "config/params")
    return parser.parse_args()
# 使用
args = get_args()
```

