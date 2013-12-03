---
layout: post
title: "Migrate Django ManyToMany field to ManyToMany through with South"
date: 2013-07-28 02:30
comments: true
categories: [Django, South, Migration, Python]

keywords: django, south, manytomany, through, migration, data import, table, orm, m2m
description: migrate django manytomany field to manytomany through with south and import old data
---

When working in a project already in production, you could encounter this case

You have a model that contains a ```ManyToMany``` field without through model, Django in this case take care itself for the join table, it create a join table that contains three columns: ID, pk of model1 and pk of model2 with a unique together constraint. At a specific time you want to add new column to the join table. Alright, with [South][1] installed you can do it easly but South doesn't import your old data it's just delete old table. How to import data from old schema to the new one?

<!-- more -->

Let's make a concret example to be able to understand this case briefly.

- Create a Django project: ```django-admin.py startproject migration_tutorial```
- Go into project: ```cd migration_tutorial```
- Create a Django app named studies: ```python manage.py startapp studies```
- In ```migration_tutorial/migration_tutorial/settings.py``` change database config to sqlite and add studies to ```INSTALLED_APPS```
- We add also South to our ```INSTALLED_APPS```

You should read [South][1] documentation if you are not familiar with. 	

Here's our changes in ```settings.py```:

{% codeblock settings.py lang:python %}

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'migration_tutorial',
        'USER': 'postgres',
        'PASSWORD': '',
        'HOST': '',
        'PORT': '',
    }
}

INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.admin',

    'south',

    'studies',
)

{% endcodeblock %}

## Let's create our model

We are going to create a simple model for Professor, Student and Course application.

- A Professor teach many Students
- A Student is taught by many Professors 

So we have a ManyToMany relation between Students and Professors. Here's our Django model under the studies app

{% codeblock models.py lang:python %}

from django.db import models


class Professor(models.Model):

	first_name = models.CharField(max_length=50)
	last_name  = models.CharField(max_length=50)

	def __unicode__(self):
		return u"%s %s" %(self.first_name, self.last_name)


class Student(models.Model):

	first_name = models.CharField(max_length=50)
	last_name  = models.CharField(max_length=50)
	professors = models.ManyToManyField(Professor)

	def __unicode__(self):
		return u"%s %s" %(self.first_name, self.last_name)

{% endcodeblock %}


## Run the project

Alright, let's run our project:

- Init South Migration: ```python manage.py schemamigration studies --init```
- Sync Database: ```python manage.py syncdb --noinput```
- Migrate our app: ```python manage.py migrate```, in this way South take care of schema creation

Now, we are going to insert some data using shell:

Enter in shell mode: ```python manage.py shell```

{% codeblock python interactive shell lang:python %}
from studies.models import Professor, Student
professor1 = Professor(first_name='Joshua', last_name='Smith')
professor1.save()
professor2 = Professor(first_name='Ethan', last_name='Brown')
professor2.save()
student1 = Student(first_name='Justin', last_name='Martin')
student1.save()
student2 = Student(first_name='Tom', last_name='Taylor')
student2.save()

#Let's add professor 1 to Student one
student1.professors.add(professor1)

#Check if data is persisted to database
student1 = Student.objects.get(first_name='Justin')
student1.professors.all()

#Let's add professor2 to student1 too
student1.professors.add(professor2)

#Professor2 to student2
student2.professors.add(professor2)

{% endcodeblock %}

Note that when working with Django ManyToManyField without through, Django provide some useful method like add method we just use.


## Our migration case

Now think if we had this model in production, we have many Students and Professors relation stored in our database. At a specific time we want to add course name taught by Professor to a student, this is a join table column. We are going no to use a through model to add this attribute.

Here's our updated models:

{% codeblock models.py lang:python %}
from django.db import models


class Professor(models.Model):

	first_name = models.CharField(max_length=50)
	last_name  = models.CharField(max_length=50)

	def __unicode__(self):
		return u"%s %s" %(self.first_name, self.last_name)


class Student(models.Model):

	first_name = models.CharField(max_length=50)
	last_name  = models.CharField(max_length=50)
	professors = models.ManyToManyField(Professor, through='Course')

	def __unicode__(self):
		return u"%s %s" %(self.first_name, self.last_name)


class Course(models.Model):

	name      = models.CharField(max_length=50)
	professor = models.ForeignKey(Professor)
	student   = models.ForeignKey(Student)

	class Meta:
		unique_together = ['professor', 'student']

	def __unicode__(self):
		return u"%s" %self.name
{% endcodeblock %}

Now, we are going to use South to migrate our schemas. Run ```python manage.py schemamigration studies --auto```.

South create a migration script under studies/migrations named ```0002_auto__add_course__add_unique_course_professor_student.py```

Here's it:

{% codeblock 0002_auto__add_course__add_unique_course_professor_student.py lang:python %}
# -*- coding: utf-8 -*-
import datetime
from south.db import db
from south.v2 import SchemaMigration
from django.db import models


class Migration(SchemaMigration):

    def forwards(self, orm):
        # Adding model 'Course'
        db.create_table(u'studies_course', (
            (u'id', self.gf('django.db.models.fields.AutoField')(primary_key=True)),
            ('name', self.gf('django.db.models.fields.CharField')(max_length=50)),
            ('professor', self.gf('django.db.models.fields.related.ForeignKey')(to=orm['studies.Professor'])),
            ('student', self.gf('django.db.models.fields.related.ForeignKey')(to=orm['studies.Student'])),
        ))
        db.send_create_signal(u'studies', ['Course'])

        # Adding unique constraint on 'Course', fields ['professor', 'student']
        db.create_unique(u'studies_course', ['professor_id', 'student_id'])

        # Removing M2M table for field professors on 'Student'
        db.delete_table(db.shorten_name(u'studies_student_professors'))


    def backwards(self, orm):
        # Removing unique constraint on 'Course', fields ['professor', 'student']
        db.delete_unique(u'studies_course', ['professor_id', 'student_id'])

        # Deleting model 'Course'
        db.delete_table(u'studies_course')

        # Adding M2M table for field professors on 'Student'
        m2m_table_name = db.shorten_name(u'studies_student_professors')
        db.create_table(m2m_table_name, (
            ('id', models.AutoField(verbose_name='ID', primary_key=True, auto_created=True)),
            ('student', models.ForeignKey(orm[u'studies.student'], null=False)),
            ('professor', models.ForeignKey(orm[u'studies.professor'], null=False))
        ))
        db.create_unique(m2m_table_name, ['student_id', 'professor_id'])


    models = {
        u'studies.course': {
            'Meta': {'unique_together': "(['professor', 'student'],)", 'object_name': 'Course'},
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            'professor': ('django.db.models.fields.related.ForeignKey', [], {'to': u"orm['studies.Professor']"}),
            'student': ('django.db.models.fields.related.ForeignKey', [], {'to': u"orm['studies.Student']"})
        },
        u'studies.professor': {
            'Meta': {'object_name': 'Professor'},
            'first_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'last_name': ('django.db.models.fields.CharField', [], {'max_length': '50'})
        },
        u'studies.student': {
            'Meta': {'object_name': 'Student'},
            'first_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'last_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            'professors': ('django.db.models.fields.related.ManyToManyField', [], {'to': u"orm['studies.Professor']", 'through': u"orm['studies.Course']", 'symmetrical': 'False'})
        }
    }

    complete_apps = ['studies']

{% endcodeblock %}

There's two main methods:

- forwards: used to apply new changes
- backwards: used to revert new changes

We are going now to add custom code to import existing relation into the new schema Course.
For the attribute name in Course I'm going to insert mathematics, in some case we need some details to know what we are going to put in the added column.

We insert our custom code in the forwards method, it load data from old schema generated by Django and push it into the new schema Course.

{% codeblock 0002_auto__add_course__add_unique_course_professor_student.py lang:python %}
# -*- coding: utf-8 -*-
from south.utils import datetime_utils as datetime
from south.db import db
from south.v2 import SchemaMigration
from django.db import models

from studies.models import Student, Professor, Course


class OldJoinMock(models.Model):

    class Meta:
        db_table = u'studies_student_professors'

    student = models.ForeignKey(Student)
    professor = models.ForeignKey(Professor)


class Migration(SchemaMigration):

    def forwards(self, orm):
        # Adding model 'Course'
        db.create_table(u'studies_course', (
            (u'id', self.gf('django.db.models.fields.AutoField')(primary_key=True)),
            ('name', self.gf('django.db.models.fields.CharField')(max_length=50)),
            ('professor', self.gf('django.db.models.fields.related.ForeignKey')(to=orm['studies.Professor'])),
            ('student', self.gf('django.db.models.fields.related.ForeignKey')(to=orm['studies.Student'])),
        ))
        db.send_create_signal(u'studies', ['Course'])

        # Adding unique constraint on 'Course', fields ['professor', 'student']
        db.create_unique(u'studies_course', ['professor_id', 'student_id'])
		
		#Our custom code here, before delete_table and after create Course table
        Course.objects.bulk_create([
            Course(
                student=data.student,
                professor=data.professor,
                name='mathematics')
            for data in OldJoinMock.objects.all()
        ])

        # Removing M2M table for field professors on 'Student'
        db.delete_table(db.shorten_name(u'studies_student_professors'))


    def backwards(self, orm):
        # Removing unique constraint on 'Course', fields ['professor', 'student']
        db.delete_unique(u'studies_course', ['professor_id', 'student_id'])

        # Deleting model 'Course'
        db.delete_table(u'studies_course')

        # Adding M2M table for field professors on 'Student'
        m2m_table_name = db.shorten_name(u'studies_student_professors')
        db.create_table(m2m_table_name, (
            ('id', models.AutoField(verbose_name='ID', primary_key=True, auto_created=True)),
            ('student', models.ForeignKey(orm[u'studies.student'], null=False)),
            ('professor', models.ForeignKey(orm[u'studies.professor'], null=False))
        ))
        db.create_unique(m2m_table_name, ['student_id', 'professor_id'])


    models = {
        u'studies.course': {
            'Meta': {'unique_together': "(['professor', 'student'],)", 'object_name': 'Course'},
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            'professor': ('django.db.models.fields.related.ForeignKey', [], {'to': u"orm['studies.Professor']"}),
            'student': ('django.db.models.fields.related.ForeignKey', [], {'to': u"orm['studies.Student']"})
        },
        u'studies.professor': {
            'Meta': {'object_name': 'Professor'},
            'first_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'last_name': ('django.db.models.fields.CharField', [], {'max_length': '50'})
        },
        u'studies.student': {
            'Meta': {'object_name': 'Student'},
            'first_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            u'id': ('django.db.models.fields.AutoField', [], {'primary_key': 'True'}),
            'last_name': ('django.db.models.fields.CharField', [], {'max_length': '50'}),
            'professors': ('django.db.models.fields.related.ManyToManyField', [], {'to': u"orm['studies.Professor']", 'through': u"orm['studies.Course']", 'symmetrical': 'False'})
        }
    }

    complete_apps = ['studies']

{% endcodeblock %}

Run migration: ```python manage.py migrate```

Let's check our data now:

{% codeblock python interactive shell lang:python %}
from studies.models import Professor, Student
student1 = Student.objects.get(first_name='Justin')
student1.professors.all() #should be professor1 and professor2
#Out: [<Professor: Joshua Smith>, <Professor: Ethan Brown>]
student2 = Student.objects.get(first_name='Tom')
student2.professors.all() #should be professor2
#Out: [<Professor: Ethan Brown>]
{% endcodeblock %}

Here's it, our relation are imported to our Course model easily. You can add also a custom code in backwards method needed for revert.

Just don't forget to update your code because methods add and remove are not valid now:

- Add: 
```python
student1.professors.add(professor1)
``` 
become 
```python
Course(student=student1, professor=professor1, name='Django')
```
- Remove: 
```python
student1.professors.remove(professor1)
```
become
```python
Course.objects.get(student=student1, professor=professor1).delete()
```

Last thing, use South in your Django projects, <b>you must use it</b> from project start else from now!

Good luck with your migrations !


[1]: http://south.readthedocs.org/en/latest/