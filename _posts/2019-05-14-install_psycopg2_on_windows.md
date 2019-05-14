---
layout: post
title:  "Install Psycopg2 on Windows"
date:   2019-05-14 19:24:20 +0800
author: bruinspaw
categories: Python PostgreSQL
---

# Step 1: Compilers Installation and configuration

https://wiki.python.org/moin/WindowsCompilers

Before do anything, install or upgrade the Setuptools Python package. It contain compatibility improvements and add automatic use of compilers:
```
pip install --upgrade setuptools
```

# Step 2: Download and install PostgreSQL

Before you continue to install python packages inside you virtualenvs download postgres itself. It contains files that are needed when compiling the psycopg2 python package. Just use the PostgreSQL installer(version 10 for example).

Important: add the postgres C:\Program Files\PostgreSQL\10\bin folder to your path. It contains the .dll needed for psycopg2.

# Step 3: Install psycopg2

```
pip install psycopg2
```

# Step 4: Test

```
import psycopg2
conn = psycopg2.connect(user='username', password='123456',
                        host='192.168.0.102', dbname='dbname')
cur = conn.cursor()
cur.execute('DROP TABLE IF EXISTS test')
cur.execute('CREATE TABLE test (data varchar)')
cur.execute("INSERT INTO test (data) VALUES ('hello')")
conn.commit()
cur.close()
conn.close()
```

