# til
A 'Today I Learned' notebook

## 1 March 2018, Thursday
In [a previous entry](Archive/2018/02#21-february-2018-wednesday), I mentioned `-v` parameter of `docker run`. While providing a local path with `-v`, it is required to provide an absolute path. So, something like this would fail:

```
# Assume I'm in my home directory and `Desktop` exists
docker run --name test -v Desktop:/Desktop -p 80:9000
```
