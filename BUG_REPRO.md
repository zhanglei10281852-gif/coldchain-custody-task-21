# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

事务回调在写入研究后返回错误，调用方收到失败，但重新查询发现研究仍已落库。请修复事务完成顺序，失败回调不得留下部分状态。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-21
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-21.git
- parent SHA：8e6ceee7cd7a4c3864468dec1998413748363b04

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-21.git bug-repro
cd bug-repro
git checkout --detach 8e6ceee7cd7a4c3864468dec1998413748363b04
go test ./internal/storage/sqlite -run "^TestTransactionRollsBackAllEntities$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestTransactionRollsBackAllEntities$" -count=1
--- FAIL: TestTransactionRollsBackAllEntities (0.07s)
    store_test.go:107: study after rollback error = <nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.077s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestTransactionRollsBackAllEntities$" -count=1
--- FAIL: TestTransactionRollsBackAllEntities (0.33s)
    store_test.go:107: study after rollback error = <nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.544s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/storage/sqlite -run ^TestTransactionRollsBackAllEntities$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
