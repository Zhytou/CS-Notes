# 内存模型

## 进程空间

## 对象生命周期

## 杂项

### new 与 malloc 的区别

| Feature               | new/delete             | malloc/free                |
| --------------------- | ---------------------- | -------------------------- |
| Memory allocated from | 'Free Store'           | 'Heap'                     |
| Return                | Fully typed pointer    | void *                     |
| On failure            | Throws                 | Returns NULL               |
| Required size         | Calculated by compiler | Must be specified in bytes |

> Technically, memory allocated by `new` comes from the 'Free Store' while memory allocated by `malloc` comes from the 'Heap'. Whether these two areas are the same is an implementation detail, which is another reason that `malloc` and `new` cannot be mixed.
