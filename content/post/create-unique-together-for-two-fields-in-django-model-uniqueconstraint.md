
+++
author = "Rogério Lima"
title = "Create unique together for two fields in django model uniqueconstraint"
date = "2023-11-10"
description = "Create unique together for two fields in django model uniqueconstraint"
tags = [
    "django",
    "orm",
    "constraint"
]
thumbnail = "images/django-bulk-inserts.png"

+++


# Create unique together for two fields in django model uniqueconstraint

### Artigo original acessado em 06/11/2023:


[Create unique together for two fields in django model uniqueconstraint](https://studygyaan.com/django/create-unique-together-for-two-fields-in-django-model-uniqueconstraint)



In Django, ensuring data integrity is essential to maintain the consistency and accuracy of your database. One common requirement is to enforce uniqueness for combinations of two or more fields in a model. While the unique_together option within Meta class is a common way to achieve this, Django 2.2 and later versions offer a more explicit approach using the UniqueConstraint option. In this blog post, will explore how to create unique constraint for two fields in Django models using UniqueConstraint.

Imagine you have a Django model representing a user profile, and you want to ensure that each user has unique combination of both their username and email address. In other words, no two users should share the same username and email. Let’s walk through the steps to achieve this using UniqueConstraint.
Step 1: Define Your Model

First, define your model in Django. Here an example of a simple user profile model:

```python
from django.db import models

class UserProfile(models.Model):
    username = models.CharField(max_length=50)
    email = models.EmailField()
    # Other fields...

Step 2: Adding UniqueConstraint

```

To ensure the uniqueness of both the username and email fields, you can use the UniqueConstraint option within the model Meta class. Modify your model as follows:

```python
from django.db import models

class UserProfile(models.Model):
    username = models.CharField(max_length=50)
    email = models.EmailField()

    class Meta:
        constraints = [
            models.UniqueConstraint(fields=['username', 'email'], name='unique_username_email')
        ]

```
In this example, we create a UniqueConstraint object within the constraints list. We specify the fields for which uniqueness should be enforced, ['username', 'email'], and provide a name for the constraint, 'unique_username_email'.
Step 3: Migrate Your Database

After adding the unique constraint, you need to create and apply a database migration to update your database schema. Use the following commands:

```bash
python manage.py makemigrations
python manage.py migrate

```
These commands will generate the necessary SQL statements to enforce the unique constraint in your database.
Step 4: Handling Violations

When you attempt to create or update a UserProfile instance with an non-unique combination of username and email, Django will raise a IntegrityError. You can handle this exception in your code to provide a meaningful response to the user.

Enforcing uniqueness for combinations of two or more fields in Django models is a crucial aspect of maintaining data integrity. Using the UniqueConstraint option, introduced in Django 2.2 and later, provides a more explicit and flexible way to create unique constraints for your model fields. This ensures that your database will reject any attempts to insert duplicate combinations of values helping you keep your data accurate and consistent.

Find this code on [Github](https://studygyaan.com/django/create-unique-together-for-two-fields-in-django-model-uniqueconstraint).
