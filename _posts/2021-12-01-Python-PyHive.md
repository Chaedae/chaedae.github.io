---
title: Python PyHive
author: ChaeDae
date: 2021-12-01 16:42:00 +0800
categories: [Python, Library]
tags: [Python, 파이썬, PyHive, hive, 하이브]
---

**PyHive**

파이썬에서 hive로 접속해서 쿼리를 할 수 있게 도와주는 라이브러리다.


pyhive 라이브러리는 Kerberos 프로토콜을 이용해서 python <-> hive 통신을 하기 때문에,
<br>
이와 관련된 라이브러리를 설치해줘야 한다.

### 설치
```
pip install sasl (인증 및 데이터보안)
pip install thrift (이기종 통신)
pip install thrift-sasl
pip install pyhive
```

### 사용
```python
from pyhive import hive

conn = hive.Connection(host='호스트명', port=포트번호, username='유저명', password='패스워드', database='데이터베이스명', auth='CUSTOM')
cur = conn.cursor()

boardNo = 3
hive_sql = f"SELECT content FROM tb_board WHERE board_no = {boardNo}"

cur.execute(hive_sql)

for result in cur.fetchall():
    print(result)

```

### 사용2 (Pandas)
```python
import pandas as pd
from pyhive import hive

conn = hive.Connection(host='호스트명', port=포트번호, username='유저명', password='패스워드', database='데이터베이스명', auth='CUSTOM')

boardNo = 3
hive_sql = f"SELECT content FROM tb_board WHERE board_no = {boardNo}"

df = pd.read_sql(hive_sql, conn)

print(df)
```
