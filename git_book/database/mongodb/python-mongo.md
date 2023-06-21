# python-mongo

基本使用

```
from pymongo import MongoClient

import pymongo

uri = "mongodb://mongo:27017/app_version"

db = MongoClient(uri).app_version
for i in db['autodeploy'].find({"Env":"pre","status" : "已提测"}).sort("_id",pymongo.DESCENDING).limit(5):

print(type(i))
```
