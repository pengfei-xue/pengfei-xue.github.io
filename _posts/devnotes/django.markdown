## How can I see the raw SQL queries Django is running?

Make sure your Django DEBUG setting is set to True. Then, just do this:

    >>> from django.db import connection
    >>> connection.queries
    [{'sql': 'SELECT polls_polls.id, polls_polls.question, polls_polls.pub_date FROM polls_polls',
    'time': '0.002'}]

connection.queries includes all SQL statements – INSERTs, UPDATES, SELECTs, etc.
Each time your app hits the database, the query will be recorded.

If you are using multiple databases, you can use the same interface on each member of the connections dictionary:

    >>> from django.db import connections
    >>> connections['my_db_alias'].queries


If you need to clear the query list manually at any point in your functions, just call reset_queries(), like this:

    from django.db import reset_queries
    reset_queries()


## how to modify OneToOneField as inline for admin console?

    class NotifyPolicyInlines(admin.StackedInline):
        model = NotifyPolicy

    class ServiceAdmin(ModelAdmin):
        filter_horizontal = ('contacts', )
        inlines = [NotifyPolicyInlines, ]

## values/values_list 查询结果只返回特定字段

Returns a QuerySet that returns dictionaries, rather than model instances, when used as an iterable.

    # This list contains a Blog object.
    >>> Blog.objects.filter(name__startswith='Beatles')
    <QuerySet [<Blog: Beatles Blog>]>

    # This list contains a dictionary.
    >>> Blog.objects.filter(name__startswith='Beatles').values()
    <QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>

    >>> Entry.objects.values_list('id').order_by('id')
    [(1,), (2,), (3,), ...]

    >>> Entry.objects.values_list('id', flat=True).order_by('id')
    [1, 2, 3, ...]
