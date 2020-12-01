学习笔记
Q: 我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

A: 根据官网文档的推荐[错误处理](http://go-database-sql.org/errors.html)方法,应该将查询为空和其他错误分开处理.
```
var name string
err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
if err != nil {
	if err == sql.ErrNoRows {
		// there were no rows, but otherwise no error occurred
	} else {
		log.Fatal(err)
	}
}
fmt.Println(name)
```

## 分情况讨论:

- 在 dao 层,直接处理,将返回值设为 NULL,nil. 这样上层比较难受,因为 err 为 nil ,每个地方都要判空.

- 直接将这个 ErrNoRow wrap 上 SQL 向上抛出.service 处理一次,不用每个地方都判 nil. 只要 err != nil 那值也不是空.

- 除了 ErrNoRow 之外的 error wrap 上 sql 语句. 直接向 service 层抛出.

## service 层处理
- 对 dao 的结果 用 errors.Is(err,sql.ErrNoRow)判,
然后执行相应逻辑即可.