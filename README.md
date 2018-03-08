# til
A 'Today I Learned' notebook

## 8 March 2018, Thursday
I needed to commit with my company email `akif@vngrs.com` in my work related git repositories. To configure the committing user in a git repository, I did following:

```
git config user.email "akif@vngrs.com"
```

This sets the email config for current git repository. After running this, the commits will be authored with my work email for this repository.

There's also `user.name` config and `--global` flag to set these globally, instead of the current git repository. Here's what it looks like:

```
git config --global user.name "Mehmet Akif Tütüncü"
```

Same name everywhere but different emails in different repositories, neat!

What if I mistakenly make a commit with wrong email? I can reset the author of a commit by:

```
git commit --amend --reset-author
```

Of course, this needs to be run after correct email is set. This will amend author info of last commit.

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
