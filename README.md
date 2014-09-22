LKDBHelper
====================================
this is sqlite ORM (an automatic database operation) <br>
thread-safe and not afraid of recursive deadlock

全面支持 NSArray，NSDictionary，ModelClass 属性了

supported NSArray, NSDictionary, ModelClass attribute

------------------------------------
Requirements
====================================

* iOS 4.3+ 
* ARC only
* FMDB(https://github.com/ccgus/fmdb)

##Adding to your project

If you are using CocoaPods, then, just add this line to your PodFile<br>

```objective-c
pod 'LKDBHelper', :head
```



##Basic usage

1 . Create a new Objective-C class for your data model

```objective-c
@interface LKTest : NSObject
@property(copy,nonatomic)NSString* name;
@property NSUInteger  age;
@property BOOL isGirl;

@property(strong,nonatomic)LKTestForeign* address;
@property(strong,nonatomic)NSArray* blah;
@property(strong,nonatomic)NSDictionary* hoho;

@property char like;
...
```
2 . in the *.m file, overwirte getTableName function  (option)

```objective-c
+(NSString *)getTableName
{
    return @"LKTestTable";
}
```
3 . in the *.m file, overwirte callback function (option)

```objective-c
@interface NSObject(LKDBHelper_Delegate)

+(void)dbDidCreateTable:(LKDBHelper*)helper tableName:(NSString*)tableName;
+(void)dbDidAlterTable:(LKDBHelper*)helper tableName:(NSString*)tableName addColumns:(NSArray*)columns;

+(BOOL)dbWillInsert:(NSObject*)entity;
+(void)dbDidInserted:(NSObject*)entity result:(BOOL)result;

+(BOOL)dbWillUpdate:(NSObject*)entity;
+(void)dbDidUpdated:(NSObject*)entity result:(BOOL)result;

+(BOOL)dbWillDelete:(NSObject*)entity;
+(void)dbDidDeleted:(NSObject*)entity result:(BOOL)result;

///data read finish
+(void)dbDidSeleted:(NSObject*)entity;

@end

```
4 . Initialize your model with data and insert to database  

```objective-c
    LKTestForeign* foreign = [[LKTestForeign alloc]init];
    foreign.address = @":asdasdasdsadasdsdas";
    foreign.postcode  = 123341;
    foreign.addid = 213214;
    
    //插入数据    insert table row
    LKTest* test = [[LKTest alloc]init];
    test.name = @"zhan san";
    test.age = 16;
    
    //外键 foreign key
    test.address = foreign;
    test.blah = @[@"1",@"2",@"3"];
    test.blah = @[@"0",@[@1],@{@"2":@2},foreign];
    test.hoho = @{@"array":test.blah,@"foreign":foreign,@"normal":@123456,@"date":[NSDate date]};
    ///warning: NSDate没做处理  所以不能在 NSArray 里 或者 NSDictionry 里面使用NSDate
    
    //异步 插入第一条 数据   Insert the first
    [test saveToDB];
    //or
    //[globalHelper insertToDB:test];
    
```
5 . select 、 delete 、 update 、 isExists 、 rowCount ...

```objective-c
    select:
        
        NSMutableArray* array = [globalHelper search:[LKTest class] where:nil orderBy:nil offset:0 count:100];
        for (NSObject* obj in array) {
            addText(@"%@",[obj printAllPropertys]);
        }
        
    delete:
        
        [globalHelper deleteToDB:test];
        
    update:
        
        test.name = "rename";
        [globalHelper updateToDB:test where:nil];
        
    isExists:
        
        [globalHelper isExistsModel:test];
    
    rowCount:
        
        [globalHelper rowCount:[LKTest class] where:nil];
        
     
```
6 . Description of parameters "where"

```objective-c
 For example: 
        single:  @"rowid = 1"                         or      @{@"rowid":@1}
 
        more:    @"rowid = 1 and sex = 0"             or      @{@"rowid":@1,@"sex":@0}
                   
                    when where is "or" type , such as @"rowid = 1 or sex = 0"
                    you only use NSString
 
        array:   @"rowid in (1,2,3)"                  or      @{@"rowid":@[@1,@2,@3]}
            
        composite:  @"rowid in (1,2,3) and sex=0 "      or      @{@"rowid":@[@1,@2,@3],@"sex":@0}
 
        If you want to be judged , only use NSString
        For example: @"date >= '2013-04-01 00:00:00'"
```

##table mapping

overwirte getTableMapping Function

```objective-c
+(NSDictionary *)getTableMapping
{
    //return nil 
    return @{@"name":LKSQLInherit,
             @"MyAge":@"age",
             @"img":LKSQLInherit,
             @"MyDate":@"date",
             @"color":LKSQLInherit,
             @"address":LKSQLUserCalculate};
}
```

##table update

```objective-c
+(void)dbDidAlterTable:(LKDBHelper *)helper tableName:(NSString *)tableName addColumns:(NSArray *)columns
{
    if([columns containsObject:@"error"])
    {
        [helper executeDB:^(FMDatabase *db) {
            NSString* sql = [NSString stringWithFormat:@"update %@ set error = name",tableName];
            [db executeUpdate:sql];
        }];
    }
    LKErrorLog(@"your know %@",columns);
}
```
## set column attribute

```objective-c
+(void)columnAttributeWithProperty:(LKDBProperty *)property
{
    if([property.sqlColumnName isEqualToString:@"MyAge"])
    {
        property.defaultValue = @"15";
    }
    if([property.propertyName isEqualToString:@"date"])
    {
        property.isUnique = YES;
        property.checkValue = @"MyDate > '2000-01-01 00:00:00'";
        property.length = 30;
    }
}
```

##demo screenshot
![demo screenshot](https://github.com/li6185377/LKDBHelper-SQLite-ORM/raw/master/screenshot/Snip20130620_8.png)
<br>table test data<br>
![](https://github.com/li6185377/LKDBHelper-SQLite-ORM/raw/master/screenshot/Snip20130620_6.png)
<br>foreign key data<br>
![](https://github.com/li6185377/LKDBHelper-SQLite-ORM/raw/master/screenshot/Snip20130620_7.png)

----------
Change-log
==========

**Version 1.1** @ 2012-6-20

- automatic table mapping
- support optional columns
- support column attribute settings
- you can return column content

**Version 1.0** @ 2013-5-19

- overwrite and rename LKDBHelper
- property type support: UIColor,NSDate,UIImage,NSData,CGRect,CGSize,CGPoint,int,float,double,NSString,short,char,bool,NSInterger..
- fix a recursive deadlock. 
- rewrite the asynchronous operation - 
- thread-safe 
- various bug modified optimize cache to improve performance 
- test and demos
- bug fixes, speed improvements

**Version 0.0.1** @ 2012-10-1

- Initial release with LKDAOBase


-------
License
=======

This code is distributed under the terms and conditions of the MIT license. 

-------
Contribution guidelines
=======

* if you are fixing a bug you discovered, please add also a unit test so I know how exactly to reproduce the bug before merging

-------
Contributors
=======

Author: Jianghuai Li

Contributors: waiting for you to join

