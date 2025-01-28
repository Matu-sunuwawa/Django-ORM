# 🚀Django-ORM

## Definition
+ Django ORM provides abstractions to work with databases, in a mostly database agnostic way.
+ It keeps “Simple things easy and hard things possible”.

<h2>1. Querying and Filtering</h2>

<img src="https://github.com/Matu-sunuwawa/Django-ORM/blob/main/django-orm.png">

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

##  How to do union of two querysets from same or different models?
+ The UNION operator is used to combine the result-set of two or more querysets.
Let’s continue with our `auth_user` model:
```
q1 = User.objects.filter(id__gte=5)
q2 = User.objects.filter(id__lte=9)

q1.union(q2)
q2.union(q1)
```
Now try this:
```
q3 = EventVillain.objects.all()
q1.union(q3)
```
output: `django.db.utils.OperationalError`
+ The union operation can be performed only with the querysets having same fields and the datatypes.

Since Hero and Villain both have the name and gender, we can use values_list to limit the selected fields then do a union.
```
Hero.objects.all().values_list("name", "gender").union(
Villain.objects.all().values_list("name", "gender"))
```

## How to select some fields only in a queryset?
Django provides two ways to do this
+ values and values_list methods on queryset.
+ only_method

CDN: Say, we want to get first_name and last_name of all the users whose name starts with R.
```
queryset = User.objects.filter(first_name__startswith='R').values('first_name', 'last_name')
```
or
```
queryset = User.objects.filter(first_name__startswith='R').only("first_name", "last_name")
```
Note That: The only difference between only and values is <mark>only also fetches the id</mark>.

## How to do a subquery expression in Django?
Let’s start with something simple:
We have a `UserParent model` which has `OnetoOne` relation with `auth user` like:
```
from django.contrib.auth.models import User
from django.db import models

class UserParent(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    parent = models.OneToOneField('self', null=True, blank=True, on_delete=models.CASCADE)
```
We will find all the UserParent which have a UserParent(this means: 'you want to find all `UserParent` entries where the <mark>parent field is not null.</mark>')
```
from django.db.models import Subquery
users = User.objects.all()
UserParent.objects.filter(user_id__in=Subquery(users.values('id')))
```
Let's continue with something more complex:
+ Bro we should jump it 😂

## How to filter a queryset with criteria based on comparing their field values
+ You can use the <mark>F object</mark>
Create some users first:
```
User.objects.create_user(email="shabda@example.com", username="shabda",first_name="Shabda", last_name="Raaj")
User.objects.create_user(email="guido@example.com", username="Guido", first_name="Guido", last_name="Guido")
```
Now you can find the users where `first_name==last_name`:
```
User.objects.filter(last_name=F("first_name"))
```
What if we wanted users whose first and last names have same letter 🤔?
+ You can set the first letter from a string using Substr("first_name", 1, 1)
Example:
```
User.objects.create_user(email="guido@example.com", username="Tim", first_name="Tim", last_name="Teters")
User.objects.annotate(first=Substr("first_name", 1, 1), last=Substr("last_name", 1, 1)).filter(first=F("last"))
```
Note That: `F` can also be used with` __gt`, `__lt` and other expressions.

## How to filter FileField without any file?
+ A FileField or ImageField stores the path of the file or image.
```
from django.db.models import Q
no_files_objects = MyModel.objects.filter(Q(file='')|Q(file=None))
```

## How to perform join operations in django ORM?
+ A SQL Join statement is used to combine data or rows from two or more tables
```
a1 = Article.objects.select_related('reporter')
```
or
```
a2 = Article.objects.filter(reporter__username='John')
```

## How to find second largest record using Django ORM ?
+ Though the ORM gives the flexibility of finding first(), last() item from the queryset
```
user = User.objects.order_by('-last_login')[1] // Second Highest record w.r.t 'last_login'

user = User.objects.order_by('-last_login')[2] // Third Highest record w.r.t 'last_login'
```

## Find rows which have duplicate field values
+ Say you want all users whose first_name matches another user
```
duplicates = User.objects.values('first_name').annotate(name_count=Count('first_name')).filter(name_count__gt=1)
```
```
duplicates
```
output: `<QuerySet [{'first_name': 'John', 'name_count': 3}]>`
If you need to fill all the records, you can do:
```
records = User.objects.filter(first_name__in=[item['first_name'] for item in duplicates])
```
```
print([item.id for item in records])
```
output: `[2, 11, 13]`

## How to find distinct field values from queryset?
+ You want to find users whose names <mark>have not been repeated</mark>. You can do this like this
```
distinct = User.objects.values('first_name').annotate(name_count=Count('first_name')).filter(name_count=1)
records = User.objects.filter(first_name__in=[item['first_name'] for item in distinct])
```
Note That:
This is different from `User.objects.distinct("first_name").all()`, which will pull up the first record when it encounters a distinct first_name

## How to group records in Django ORM?
+ Grouping of records in Django ORM can be done using aggregation functions like Max, Min, Avg, Sum.
```
from django.db.models import Avg, Max, Min, Sum, Count

User.objects.all().aggregate(Avg('id'))
User.objects.all().aggregate(Max('id'))
User.objects.all().aggregate(Min('id'))
User.objects.all().aggregate(Sum('id'))
```
output:
```
{'id__avg': 7.571428571428571}
{'id__max': 15}
{'id__min': 1}
{'id__sum': 106}
```

## How to efficiently select a random object from a model?
Example:
```
class Category(models.Model):
  name = models.CharField(max_length=100)

  class Meta:
    verbose_name_plural = "Categories"
  def __str__(self):
    return self.name
```
solution:
```
def get_random():
  return Category.objects.order_by("?").first()
```
Better solution(instead of sorting the whole table, you can get the max id, generate a random number in range [1, max_id], and filter that)
+ You are assuming that there have been no deletions.
```
from django.db.models import Max
from entities.models import Category

import random

def get_random2():
    max_id = Category.objects.all().aggregate(max_id=Max("id"))['max_id']
    pk = random.randint(1, max_id)
    return Category.objects.get(pk=pk)
```
If your models has deletions:
```
def get_random3():
    max_id = Category.objects.all().aggregate(max_id=Max("id"))['max_id']
    while True:
        pk = random.randint(1, max_id)
        category = Category.objects.filter(pk=pk).first()
        if category:
            return category
```
compare time complexity:
```
timeit.timeit(get_random3, number=100)
timeit.timeit(get_random, number=100)
```
output:
```
0.20055226399563253
56.92513192095794
```

<h2>2. Creating, Updating and Deleting things</h2>

## How to create multiple objects in one shot?
+ There are conditions when we want to save multiple objects in one go.
```
Category.objects.all().count()

Category.objects.bulk_create([Category(name="God"),Category(name="Demi God"),Category(name="Mortal")])

Category.objects.all().count()
```
Note That: `bulk_create` takes a list of unsaved objects.

## How to copy or clone an existing model object?
+ There is no built-in method for copying model instances, it is possible to create new instance with all fields values copied.
+ If an instance is saved with instance’s pk set to None, the instance is used to create a new record in the DB. That means every field other than the PK is copied.
```
Hero.objects.all().count()
hero = Hero.objects.first()
hero.pk = None
hero.save()
Hero.objects.all().count()
```
output:
```
4
5
```

## How to ensure that only one object can be created?
Bro <mark>LOCKING</mark>:
Sometimes you want to ensure that only one record can be created for a model. This is commonly required as application configuration store, or as a `locking mechanism` to access shared resources.
```
class Origin(models.Model):
  name = models.CharField(max_length=100)

  def save(self, *args, **kwargs):
    if self.__class__.objects.count():
      self.pk = self.__class__.objects.first().pk
    super().save(*args, **kwargs)
```

## How to update denormalized fields in other models on save?
You have models like this:
```
class Category(models.Model):
  name = models.CharField(max_length=100)
  hero_count = models.PositiveIntegerField()
  villain_count = models.PositiveIntegerField()
  class Meta:
    verbose_name_plural = "Categories"

class Hero(models.Model):
  name = models.CharField(max_length=100)
  category = models.ForeignKey(Category, on_delete=models.CASCADE)

class Villain(models.Model):
  name = models.CharField(max_length=100)
  category = models.ForeignKey(Category, on_delete=models.CASCADE)
```
+ You need the `hero_count` and `villain_count`, to be updated when new objects are created. You can do something like this:
```
class Hero(models.Model):
  # ...
  def save(self, *args, **kwargs):
    if not self.pk:
      Category.objects.filter(pk=self.category_id).update(hero_count=F('hero_count')+1)
    super().save(*args, **kwargs)

class Villain(models.Model):
  #...
  def save(self, *args, **kwargs):
    if not self.pk:
      Category.objects.filter(pk=self.category_id).update(villain_count=F('villain_count')+1)
    super().save(*args, **kwargs)
```

## How to perform truncate like operation using Django ORM?
+ Django doesn’t provide a builtin to truncate a table, but still similar result can be achived using `delete() method`.
```
Category.objects.all().count()
Category.objects.all().delete()
Category.objects.all().count()
```
output:
```
7
(7, {'entity.Category': 7})
0
```
or You can add a classmethod to Category if you want to enable truncate.
```
class Category(models.Model):
  # ...
  @classmethod
  def truncate(cls):
    with connection.cursor() as cursor:
      cursor.execute('TRUNCATE TABLE "{0}" CASCADE'.format(cls._meta.db_table))
```
*Then you can call Category.truncate()

## What signals are raised by Django during object creation or update?
Django provides signals which allows hooking into a model objects creation and deletion lifecycle.
+ pre_init
+ post_init
+ pre_save
+ post_save
+ pre_delete
+ post_delete

<h2>3. Ordering things</h2>

## How to order a queryset in ascending or descending order?
+ Ordering of the queryset can be achieved by order_by method
```
User.objects.all().order_by('date_joined') # For ascending
User.objects.all().order_by('-date_joined') # For descending;
```
You can pass multiple fields to order_by:
```
User.objects.all().order_by('date_joined', '-last_login')
```











