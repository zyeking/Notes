# go-gin-example 10： GORM-Callback

# 定制GORM Callbacks

## GORM Callback 是什么
> You could define callback methods to pointer of model struct, it will be called when creating, updating, querying, deleting, if any callback returns an error, gorm will stop future operations and rollback all changes.

可以给模型结构体指针定义回调函数, 它将会在被创建/更新/查询/删除的时候调用, 如果回调返回了错误, gorm会停止未来行为操作并且回退所有改变. 

---
之前程序未实现`Callback`方法, 需要为所有文件单独写一次`BeforeCreate`、`BeforeUpdate`方法
![](http://q1hfuy9re.bkt.clouddn.com/20191206003247.png)
![](http://q1hfuy9re.bkt.clouddn.com/20191206003400.png)

## 使用
### gorm支持的callback方法
+ 创建：BeforeSave、BeforeCreate、AfterCreate、AfterSave
+ 更新：BeforeSave、BeforeUpdate、AfterUpdate、AfterSave
+ 删除：BeforeDelete、AfterDelete
+ 查询：AfterFind

### 定义callback
1. 在`model.go`文件中定义
```go
// updateTimeStampForCreateCallback will set `CreatedOn`, `ModifiedOn` when creating
func updateTimeStampForCreateCallback(scope *gorm.Scope) {
    if !scope.HasError() {
        nowTime := time.Now().Unix()
        if createTimeField, ok := scope.FieldByName("CreatedOn"); ok {
            if createTimeField.IsBlank {
                createTimeField.Set(nowTime)
            }
        }

        if modifyTimeField, ok := scope.FieldByName("ModifiedOn"); ok {
            if modifyTimeField.IsBlank {
                modifyTimeField.Set(nowTime)
            }
        }
    }
}

// updateTimeStampForUpdateCallback will set `ModifyTime` when updating
func updateTimeStampForUpdateCallback(scope *gorm.Scope) {
    if _, ok := scope.Get("gorm:update_column"); !ok {
        scope.SetColumn("ModifiedOn", time.Now().Unix())
    }
}
```
1. 通过`scope.FieldByName(name)`判断是否存在相关字段, 该方法通过`scope.Fields()`获取所有字段
2. 通过`.IsBlank`判断值是否为空
3. 通过`.Set(interface{})`设置相关值
4. 通过`scope.Get()`获取参数的参数值, 案例中回去查找`gorm:update_column`这个字段的属性
5. 通过`scope.SetColumn(Field, value)`设定字段的值

### 调用callback
1. 在`model.go`的`ini`函数中注册callback
```go
db.Callback().Create().Replace("gorm:update_time_stamp", updateTimeStampForCreateCallback)
db.Callback().Update().Replace("gorm:update_time_stamp", updateTimeStampForUpdateCallback)
```
## 效果
当程序写了`Callback`方法的时候, 当GORM执行到相关的操作会自动触发相应的`Callback`方法

## 拓展
1. 软删除, 添加删除时间, 为`model.go`的`Model`结构体添加`DeletedOn`字段
```go
type Model struct {
    ID int `gorm:"primary_key" json:"id"`
    CreatedOn int `json:"created_on"`
    ModifiedOn int `json:"modified_on"`
    DeletedOn int `json:"deleted_on"`
}
```

```go
func deleteCallback(scope *gorm.Scope) {
    if !scope.HasError() {
        var extraOption string
        if str, ok := scope.Get("gorm:delete_option"); ok {
            extraOption = fmt.Sprint(str)
        }

        deletedOnField, hasDeletedOnField := scope.FieldByName("DeletedOn")

        if !scope.Search.Unscoped && hasDeletedOnField {
            scope.Raw(fmt.Sprintf(
                "UPDATE %v SET %v=%v%v%v",
                scope.QuotedTableName(),
                scope.Quote(deletedOnField.DBName),
                scope.AddToVars(time.Now().Unix()),
                addExtraSpaceIfExist(scope.CombinedConditionSql()),
                addExtraSpaceIfExist(extraOption),
            )).Exec()
        } else {
            scope.Raw(fmt.Sprintf(
                "DELETE FROM %v%v%v",
                scope.QuotedTableName(),
                addExtraSpaceIfExist(scope.CombinedConditionSql()),
                addExtraSpaceIfExist(extraOption),
            )).Exec()
        }
    }
}

func addExtraSpaceIfExist(str string) string {
    if str != "" {
        return " " + str
    }
    return ""
}
```
在`model.go`的`ini函数`中添加`db.Callback().Delete().Replace("gorm:delete", deleteCallback)`
1. `scope.QuotedTableName()`返回引用的表名
2. `scope.Raw()`构建原生sql
3. `fmt.Sprintf()`格式化并且返回格式化后的字符串数据
4. `scope.AddToVars(value)`为字段添加参数
5. `scope.Quote()`转义
6. `scope`当你在数据库中文完成任何操作, scope包含了当前操作信息
```go
// Scope contain current operation's information when you perform any operation on the database
type Scope struct {
	Search          *search
	Value           interface{}
	SQL             string
	SQLVars         []interface{}
	db              *DB
	instanceID      string
	primaryKeyField *Field
	skipLeft        bool
	fields          *[]*Field
	selectAttrs     *[]string
}
```

## REF:
[official: callback in gorm](http://gorm.book.jasperxu.com/callbacks.html)