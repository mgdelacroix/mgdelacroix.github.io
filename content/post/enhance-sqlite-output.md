+++
date = "2017-04-16"
draft = false
title = "Enhance SQLite output"
slug = "enhance-sqlite-output"
tags = ["sqlite", "trick", "sysadmin"]
author = "Miguel de la Cruz"
+++

I've been doing some development in [Baoqu](https://github.com/baoqu) with [SQLite](https://www.sqlite.org) recently, and one of the problems I've faced is that the default output from a query in the SQLite client is too basic, almost confusing:

```sql
sqlite> select * from users;
1|Ramira
2|Maria
3|Yamilo
4|Mario
5|Pérez-Reverte
6|Yisus
7|Miguel
8|Marta
9|andy
10|lol
11|lol2
12|lolaso
```

That's a simple query, let's complicate it a little more:

```sql
sqlite> select * from users as u
          inner join users_circles as uc
            on u.id=uc."user-id"
          inner join circles as c
            on uc."circle-id"=c.id;
1|Ramira|1|1|1|1|1|3|3
2|Maria|2|1|1|1|1|3|3
3|Yamilo|3|1|1|1|1|3|3
4|Mario|4|2|2|1|1|3|3
5|Pérez-Reverte|5|2|2|1|1|3|3
6|Yisus|6|2|2|1|1|3|3
1|Ramira|1|3|3|1|2|3|9
2|Maria|2|3|3|1|2|3|9
3|Yamilo|3|3|3|1|2|3|9
4|Mario|4|3|3|1|2|3|9
5|Pérez-Reverte|5|3|3|1|2|3|9
6|Yisus|6|3|3|1|2|3|9
7|Miguel|7|4|4|1|1|3|3
8|Marta |8|4|4|1|1|3|3
9|andy|9|4|4|1|1|3|3
10|lol|10|5|5|1|1|3|
11|lol2|11|5|5|1|1|3|
7|Miguel|7|3|3|1|2|3|9
8|Marta |8|3|3|1|2|3|9
9|andy|9|3|3|1|2|3|9
12|lolaso|12|5|5|1|1|3|
1|Ramira|1|9|9|1|3|3|
2|Maria|2|9|9|1|3|3|
3|Yamilo|3|9|9|1|3|3|
4|Mario|4|9|9|1|3|3|
5|Pérez-Reverte|5|9|9|1|3|3|
6|Yisus|6|9|9|1|3|3|
7|Miguel|7|9|9|1|3|3|
8|Marta |8|9|9|1|3|3|
9|andy|9|9|9|1|3|3|
```

That is simply unreadable, and not only because almost all the columns are plain integers, but the untabbed layout and the lack of column names only makes it worst.

Luckly, the SQLite client provides us with some options we can activate to improve this. In the client, run:

```sql
sqlite> .headers on
sqlite> .mode column
```

And now, the output is much more readable:

```sql
sqlite> select * from users as u
          inner join users_circles as uc
            on u.id=uc."user-id";
id          name        user-id     circle-id
----------  ----------  ----------  ----------
1           Ramira      1           1
2           Maria       2           1
3           Yamilo      3           1
4           Mario       4           2
5           Perez-Rev   5           2
6           Yisus       6           2

...
```
You can use the `.width` option to adjust each column width:

```sql
sqlite> .width 2 14 7 9
sqlite> select * from users as u
          inner join users_circles as uc
            on u.id=uc."user-id";
id  name            user-id  circle-id
--  --------------  -------  ---------
1   Ramira          1        1
2   Maria           2        1
3   Yamilo          3        1
4   Mario           4        2
5   Perez-Reverte   5        2
6   Yisus           6        2

...
```

If you want this commands to run each time you use the SQLite client, just write them to a `$HOME/.sqliterc` file. You can find more info about this commands at [their section in the SQLite documentation](https://www.sqlite.org/cli.html#special_commands_to_sqlite3_dot_commands_).
