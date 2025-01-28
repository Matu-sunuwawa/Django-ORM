# 🚀Django-ORM

## Definition
+ Django ORM provides abstractions to work with databases, in a mostly database agnostic way.
+ It keeps “Simple things easy and hard things possible”.

## How to find the query associated with a queryset?
+ Youn can get str of any queryset.query to get the sql.
For example: You have a model called `Event`. For getting all records, you will write something like `Event.objects.all()`,
             then do `str(queryset.query)`, so by doing such stuff you can get the sql like:
```
SELECT "events_event"."id", "events_event"."epic_id",
"events_event"."details", "events_event"."years_ago"
FROM "events_event"
```
Let's make it more fun:
+ You want to retrieve some objects from the database and filter them based on certain conditions. Naturally, you would need to write some SQL commands like:
```
SELECT "events_event"."id", "events_event"."epic_id", "events_event"."details",
"events_event"."years_ago" FROM "events_event"
WHERE "events_event"."years_ago" > 5
```
But you do not have to do this in order to get objects based on some conditions. Instead, you can achieve this within your application (Django) simply by:
```
queryset = Event.objects.filter(years_ago__gt=5)
str(queryset.query)
```
🎉All done! That is it bro.

## How to do <mark>OR</mark> queries in Django ORM?
+ django.contrib.auth have a table called `auth_user`: username, first_name, last_name and more.
CDN: Say you want find all users with firstname starting with ‘R’ and last_name starting with ‘D’.
### The query in detail
SQL:
```
SELECT username, first_name, last_name, email FROM auth_user WHERE first_name LIKE 'R%' OR last_name LIKE 'D%';
```
Similarly `our ORM query` would look like:
```
queryset = User.objects.filter( first_name__startswith='R') | User.objects.filter(last_name__startswith='D')
```
or
```
from django.db.models import Q
qs = User.objects.filter(Q(first_name__startswith='R')|Q(last_name__startswith='D'))
```

<h4>That is my ORM ... I call it Real Definition of ORM ... or abstraction(helps to work with database) provided by ORM</h4>

`queryset`:
```
<QuerySet [<User: Ricky>, <User: Ritesh>, <User: Radha>, <User: Raghu>, <User: rishab>]>
```
## How to do <mark>AND</mark> queries in Django ORM?
### The query in detail
SQL:
```
SELECT username, first_name, last_name, email FROM auth_user WHERE first_name LIKE 'R%' AND last_name LIKE 'D%';
```
ORM query:
+ The default way to combine multiple conditions in filter is <mark>AND</mark>, so you can just do.
```
queryset_1 = User.objects.filter(first_name__startswith='R',last_name__startswith='D')
```
or
```
queryset_2 = User.objects.filter(first_name__startswith='R') & User.objects.filter(last_name__startswith='D')
```
or
```
from django.db.models import Q
queryset_3 = User.objects.filter(Q(first_name__startswith='R') &Q(last_name__startswith='D'))
```
verify that they are all same RUN:
```
str(queryset_2.query)
```
```
str(queryset_1.query) == str(queryset_2.query) == str(queryset_3.query)
```
output:
`True`

## How to do a NOT query in Django queryset?
### The query in detail
SQL:
```
SELECT id, username, first_name, last_name, email FROM auth_user WHERE NOT id < 5;
```
Django provides two options.
+ exclude(<condition>)
+ filter(~Q(<condition>))
```
queryset = User.objects.exclude(id__lt=5)
```
or
```
from django.db.models import Q
queryset = User.objects.filter(~Q(id__lt=5))
```
























