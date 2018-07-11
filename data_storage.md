# 数据存储

## 1. iOS数据存储的几种方式
- 沙盒
- Plist存储
- Preference（偏好设置）
- NSKeyedArchiver归档 / NSKeyedUnarchiver解档
- SQLite3的使用
- FMDB
- Core Data

## 应用沙盒
- 每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒

- 应用沙盒的文件系统目录，如下图所示（假设应用的名称叫Layer）