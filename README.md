# til
A 'Today I Learned' notebook

## 6 March 2018, Tuesday
Elasticsearch has a `_cat` API that you can use to list many stuff in different formats. For what it can list, check out [docs](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/cat.html). Here are some examples:

```
> http :9200/_cat

/_cat/shards
/_cat/shards/{index}
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/health
...
```

```
> http :9200/_cat/health v==true

epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1520346508 14:28:28  elasticsearch yellow          1         1     30  30    0    0       30             0                  -                 50.0%
```

```
> http :9200/_cat/indices format==json

[
    {
        "docs.count": "10",
        "docs.deleted": "0",
        "health": "yellow",
        "index": "test",
        "pri": "5",
        "pri.store.size": "81.5kb",
        "rep": "1",
        "status": "open",
        "store.size": "81.5kb",
        "uuid": "o1sPmYjjTMS0YbhmmFjvPg"
    }
]
```

## 1 March 2018, Thursday
In [a previous entry](Archive/2018/02#21-february-2018-wednesday), I mentioned `-v` parameter of `docker run`. While providing a local path with `-v`, it is required to provide an absolute path. So, something like this would fail:

```
# Assume I'm in my home directory and `Desktop` exists
docker run --name test -v Desktop:/Desktop -p 80:9000
```
