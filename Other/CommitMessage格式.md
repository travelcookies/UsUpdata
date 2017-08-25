# Commit Message 格式

<type>(<scope>): <subject>

## Header
Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。

type

用于说明 commit 的类别，只允许使用下面7个标识。

```
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
如果type为feat和fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。
```


##scope

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在Angular，可以是$location, $browser, $compile, $rootScope, ngHref, ngClick, ngView等。

如果你的修改影响了不止一个scope，你可以使用*代替。

## subject

subject是 commit 目的的简短描述，不超过50个字符。

## 其他注意事项：

以动词开头，使用第一人称现在时，比如change，而不是changed或changes
第一个字母小写
结尾不加句号（.）
