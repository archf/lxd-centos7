# Building LXC from source.

Erase OS packages fist:

```bash
yum remove $(rpm -qa  | grep -i -I lxc)
```

Using the provided Makefile:

```bash
make all
```
