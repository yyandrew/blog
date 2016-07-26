---
layout: post
title: "pyqt4 learning"
date: 2016-07-17 08:56:06 +0800
comments: true
categories: "Python"
---
1,获取QTableView中被选中的行

```python
indexes = table.selectionModel().selectedRows()
for index in sorted(indexes):
    print('Row %d is selected' % index.row())
```

2,怎么删除QTableView中的行，分两步。step1，触发`removeRows`方法；step2，重新实现`QtCore.QAbstractTableModel::removeRows`方法。

```python
# 触发removeRows方法，其中index来自上面的代码
row = index.row()
table.model().removeRows(row, 2) # 删除从第row行开始的后2列

# 重新实现removeRows(必须，否则行不能被删除)
class PasswordModel(QtCore.QAbstractTableModel):
    def removeRows(self, position, rows=1, index=QtCore.QModelIndex()):
        self.beginRemoveRows(QtCore.QModelIndex(), position, position+rows-1)
        for row in range(rows):
            del self.pw_data[position]
        self.endRemoveRows()
        return True
```
