# 容器挂载目录写不进去 → 用 `chown`，别用 `chmod 777`

**症状**

容器里能看到挂载目录，`ls` 正常，一写就报：

```
touch: cannot touch '/srv/betalog/x': Permission denied
```

**为什么**

容器里跑的是 uid 1000，宿主机目录属主是 `root:root`。两边是同一个内核、同一套 uid，只是名字不同：

```
容器里   uid 1000 → jenkins
宿主机   uid 1000 → alpha
```

**解法**

```bash
chown 1000:1000 /srv/betalog
```

验证，属主应该从 `root root` 变成 `alpha alpha`：

```bash
ls -ld /srv/betalog
```

**别用 `chmod 777`**

能通，但等于把门拆了——机器上任何用户、任何进程都能往里写。

**记住**

> 权限问题先问一句：**"该给谁"**，而不是 **"给多大"**。
> `chown` 回答第一个，`chmod 777` 逃避第二个。

---

*2026-11-19 · Jenkins 挂载 `/srv/betalog` 踩的*
