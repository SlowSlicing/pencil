* 使用 Label

　　我这里给 `Service` 和 `Pod` 都新增一个 `Label` :

| key | value | describe |
| --- | --- | --- |
| active | `running` | 正常运行的 Pod |
| active | `debugging` | 调试排除故障的 Pod |
| ...... | ...... | ...... |

　　在 Svrvice 和 Pod 初始启动之后 Label `active` 都是 `running`，如果需要调试，可以把单个 Pod 的 active 更改为 `debugging`。

```
$ kubectl label pod <pod-name> --overwrite status=debuging
```

　　然后就可以愉快的调试了。

> 　　如果想恢复，简单，再改回去 `running` 即可。