# Django 4.2.x sqlite bug demonstration

Django 4.2.7 (latest stable) has a bug, that can not handle migrations with table comments.
It also works with dev version (Django==5.0rc1) and current master branch
(`git+https://github.com/django/django.git@c9ce764f59c1e809b210337980ae10c4b1d0f9be#egg=Django
`).

Sqlite itself does not support neither table comments, nor field comments, but 
`django.db.backends.sqlite3.schema.DatabaseSchemaEditor` does not think so and generates
COMMENT sql for it

Creation of model is not affected by this bug (`proj/question/migrations/0001_initial.py` works perfectly)

Changing of model `Meta.db_table_comment` throws an exception (`proj/question/migrations/0002_alter_questionthree_table_comment.py` is affected)

## Versions

```shell
$sqlite3 --version
3.37.2 2022-01-06 13:25:41 872ba256cbf61d9290b571c0e6d82a20c224ca3ad82971edc46b29818d5dalt1
```

```shell
$pip freeze 
asgiref==3.7.2
Django==4.2.7
sqlparse==0.4.4
typing_extensions==4.8.0
```

```shell
$lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.3 LTS
Release:        22.04
Codename:       jammy
```

## Steps to reproduce

1. Clone this repo
2. (optional) create a virtualenv (`python3 -m venv ./env`) and execute it (`source ./env/bin/activate`)
3. install requirements (`pip install requirements.txt`)
4. Try to migrate this application (it defaults to sqlite database) (`cd proj; ./manage.py migrate`)

After this you should see this traceback:

```shell
$./manage.py migrate
System check identified some issues:

WARNINGS:
question.QuestionThree: (models.W046) SQLite does not support comments on tables (db_table_comment).
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, question, sessions
Running migrations:
  Applying question.0001_initial... OK
  Applying question.0002_alter_questionthree_table_comment...Traceback (most recent call last):
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 89, in _execute
    return self.cursor.execute(sql, params)
  File "env/lib/python3.10/site-packages/django/db/backends/sqlite3/base.py", line 328, in execute
    return super().execute(query, params)
sqlite3.OperationalError: near "COMMENT": syntax error

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "proj/./manage.py", line 22, in <module>
    main()
  File "proj/./manage.py", line 18, in main
    execute_from_command_line(sys.argv)
  File "env/lib/python3.10/site-packages/django/core/management/__init__.py", line 442, in execute_from_command_line
    utility.execute()
  File "env/lib/python3.10/site-packages/django/core/management/__init__.py", line 436, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "env/lib/python3.10/site-packages/django/core/management/base.py", line 412, in run_from_argv
    self.execute(*args, **cmd_options)
  File "env/lib/python3.10/site-packages/django/core/management/base.py", line 458, in execute
    output = self.handle(*args, **options)
  File "env/lib/python3.10/site-packages/django/core/management/base.py", line 106, in wrapper
    res = handle_func(*args, **kwargs)
  File "env/lib/python3.10/site-packages/django/core/management/commands/migrate.py", line 356, in handle
    post_migrate_state = executor.migrate(
  File "env/lib/python3.10/site-packages/django/db/migrations/executor.py", line 135, in migrate
    state = self._migrate_all_forwards(
  File "env/lib/python3.10/site-packages/django/db/migrations/executor.py", line 167, in _migrate_all_forwards
    state = self.apply_migration(
  File "env/lib/python3.10/site-packages/django/db/migrations/executor.py", line 252, in apply_migration
    state = migration.apply(state, schema_editor)
  File "env/lib/python3.10/site-packages/django/db/migrations/migration.py", line 132, in apply
    operation.database_forwards(
  File "env/lib/python3.10/site-packages/django/db/migrations/operations/models.py", line 610, in database_forwards
    schema_editor.alter_db_table_comment(
  File "env/lib/python3.10/site-packages/django/db/backends/base/schema.py", line 640, in alter_db_table_comment
    self.execute(
  File "env/lib/python3.10/site-packages/django/db/backends/base/schema.py", line 201, in execute
    cursor.execute(sql, params)
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 102, in execute
    return super().execute(sql, params)
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 67, in execute
    return self._execute_with_wrappers(
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 80, in _execute_with_wrappers
    return executor(sql, params, many, context)
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 84, in _execute
    with self.db.wrap_database_errors:
  File "env/lib/python3.10/site-packages/django/db/utils.py", line 91, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "env/lib/python3.10/site-packages/django/db/backends/utils.py", line 89, in _execute
    return self.cursor.execute(sql, params)
  File "env/lib/python3.10/site-packages/django/db/backends/sqlite3/base.py", line 328, in execute
    return super().execute(query, params)
django.db.utils.OperationalError: near "COMMENT": syntax error
```
