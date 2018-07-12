# FMDB
---
FMDB封装了SQLite的C语言API，更加面向对象。
首先需要明确的是FMDB中的三个类。

- FMDatabase：可以理解成一个数据库。

- FMResultSet：查询的结果集合。

- FMDatabaseQueue：运用多线程，可执行多个查询、更新。线程安全。

## FMDB基本语法
查询
```objectivec
[db executeQuery:@"select id, name, age from t_person"]
```
更新
```objectivec
// CREATE, UPDATE, INSERT, DELETE, DROP，都使用executeUpdte
[db executeUpdate:@"create table if not exists t_person (id integer primary key autoincrement, name text, age integer)"]
```
## FMDB的基本使用
在项目中导入FMDB框架和sqlite3.0.tbd，导入头文件。

### 1. 打开数据库，并创建表

```objectivec
#import "ViewController.h"
#import <FMDB.h>

@interface ViewController ()

@end

@implementation ViewController{
    FMDatabase *db;
}

- (void)openCreateDB {

    //存放数据的路径
    NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
    NSString *filePath = [path stringByAppendingPathComponent:@"person.sqlite"];
    
    //初始化FMDatabase
    db = [FMDatabase databaseWithPath:filePath];
    
    //打开数据库并创建person表，person中有主键id，姓名name，年龄age
    if ([db open]) {
        BOOL success = [db executeUpdate:@"create table if not exists t_person (id integer primary key autoincrement, name text, age integer)"];
        if (success) {
            NSLog(@"创表成功");
        }else {
            NSLog(@"创建表失败");
        }
    } else {
        NSLog(@"打开失败");
    }
}
```

### 2. 插入数据

```objectivec
-(void)insertData {
    BOOL success = [db executeUpdate:@"insert into t_person(name,age) values(?,?)",@"jack",@17];
    if (success) {
        NSLog(@"添加数据成功");
    } else {
        NSLog(@"添加数据失败");
    }
}
```
### 3. 删除数据

```objectivec
-(void)deleteData {
    BOOL success = [db executeUpdate:@"delete from t_person where name = 'lily'"];
    if (success) {
        NSLog(@"删除数据成功");
    } else {
        NSLog(@"删除数据失败");
    }
}
```
### 4. 修改数据
```objectivec
-(void)updateData {
    BOOL success = [db executeUpdate:@"update t_person set name = 'lily' where age = 17"];
    if (success) {
        NSLog(@"更新数据成功");
    } else {
        NSLog(@"更新数据失败");
    }
}
```
### 5. 查询数据

执行查询语句，用FMResultSet接收查询结果：FMResultSet *set = [db executeQuery:@"select id, name, age from t_person"]

遍历查询结果：[set next]

拿到每条数的姓名：NSString *name = [set stringForColumnIndex:1];

也可以这样拿到每条数据的姓名：NSString *name = [result stringForColumn:@"name"];

```objectivec
FMResultSet *set = [db executeQuery:@"select id, name, age from t_person"];
while ([set next]) {
    int ID = [set intForColumnIndex:0];
    NSString *name = [set stringForColumnIndex:1];
    int age = [set intForColumnIndex:2];
    NSLog(@"%d,%@,%d",ID,name,age);
}
```
6. 删除表

删除指定表：[db executeUpdate:@"drop table if exists t_person"]

-(void)dropTable {
    BOOL success = [db executeUpdate:@"drop table if exists t_person"];
    if (success) {
        NSLog(@"删除表成功");
    } else {
        NSLog(@"删除表失败");
    }
}
FMDatabaseQueue基本使用
FMDatabase是线程不安全的，当FMDB数据存储想要使用多线程的时候，FMDatabaseQueue就派上用场了。

初始化FMDatabaseQueue的方法与FMDatabase类似

//数据文件路径
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSString *filePath = [path stringByAppendingPathComponent:@"student.sqlite"];
    
//初始化FMDatabaseQueue
FMDatabaseQueue *dbQueue = [FMDatabaseQueue databaseQueueWithPath:filePath];
在FMDatabaseQueue中执行命令的时候也是非常方便，直接在一个block中进行操作

-(void)FMDdatabaseQueueFunction {
    
    NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
    //数据文件路径
    NSString *filePath = [path stringByAppendingPathComponent:@"student.sqlite"];
    //初始化FMDatabaseQueue
    FMDatabaseQueue *dbQueue = [FMDatabaseQueue databaseQueueWithPath:filePath];
    
    //在block中执行SQLite语句命令
    [dbQueue inDatabase:^(FMDatabase * _Nonnull db) {
        //创建表
        [db executeUpdate:@"create table if not exists t_student (id integer primary key autoincrement, name text, age integer)"];
        //添加数据
        [db executeUpdate:@"insert into t_student(name,age) values(?,?)",@"jack",@17];
        [db executeUpdate:@"insert into t_student(name,age) values(?,?)",@"lily",@16];
        //查询数据
        FMResultSet *set = [db executeQuery:@"select id, name, age from t_student"];
        //遍历查询到的数据
        while ([set next]) {
            int ID = [set intForColumn:@"id"];
            NSString *name = [set stringForColumn:@"name"];
            int age = [set intForColumn:@"age"];
            NSLog(@"%d,%@,%d",ID,name,age);
        }
        
    }];
}
FMDB中的事务
什么是事务？

事务（Transaction）是不可分割的一个整体操作，要么都执行，要么都不执行。
举个例子，幼儿园有20位小朋友由老师组织出去春游，返校的时候，所有人依次登上校车，这时候如果有一位小朋友没有上车，车也是不能出发的。所以哪怕19人都上了车，也等于0人上车。20人是一个整体。
当然这个例子可能不是很精准。

FMDB中有事务的回滚操作，也就是说，当一个整体事务在执行的时候出了一点小问题，则执行回滚，之后这套事务中的所有操作将整体无效。

下面代码中，利用事务循环向数据库中添加2000条数据，假如在添加的过程中出现了一些问题，由于执行了*rollback = YES的回滚操作，数据库中一个数据都不会出现。
如果第2000条数据的添加出了问题，哪怕之前已经添加了1999条数据，由于执行了回滚，数据库中依然一个数据都没有。

//数据库路径
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSString *filePath = [path stringByAppendingPathComponent:@"student.sqlite"];

//初始化FMDatabaseQueue
FMDatabaseQueue *dbQueue = [FMDatabaseQueue databaseQueueWithPath:filePath];

//FMDatabaseQueue的事务inTransaction
[dbQueue inTransaction:^(FMDatabase * _Nonnull db, BOOL * _Nonnull rollback) {
        //创建表
        [db executeUpdate:@"create table if not exists t_student (id integer primary key autoincrement, name text, age integer)"];
        //循环添加2000条数据
        for (int i = 0; i < 2000; i++) {
            BOOL success = [db executeUpdate:@"insert into t_student(name,age) values(?,?)",@"jack",@(i)];
            //如果添加数据出现问题，则回滚
            if (!success) {
                //数据回滚
                *rollback = YES;
                return;
            }
        }
    }];

作者：remember17
链接：https://www.jianshu.com/p/e88880be794f
來源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。