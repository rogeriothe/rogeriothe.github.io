+++
author = "Rogério Lima"
title = "Bulk insert com Django"
date = "2023-10-26"
description = "How to Use Django Bulk Inserts for Greater Efficiency"
tags = [
    "django",
    "orm",
]
thumbnail = "images/django-bulk-inserts.jpg"

+++


How to Use Django Bulk Inserts for Greater Efficiency


CEO Tobias McNulty is programming at his desk.

It's been awhile since we last discussed bulk inserts on the Caktus blog. The idea is simple: if you have an application that needs to insert a lot of data into a Django model — for example a background task that processes a CSV file (or some other text file) — it pays to "chunk" those updates to the database so that multiple records are created through a single database operation. This reduces the total number of round-trips to the database, something my colleague Dan Poirier discussed in more detail in the post linked above.

Today, we use Django's Model.objects.bulk_create() regularly to help speed up operations that insert a lot of data into a database. One of those projects involves processing a spreadsheet with multiple tabs, each of which might contain thousands or even tens of thousands of records, some of which might correspond to multiple model classes. We also need to validate the data in the spreadsheet and return errors to the user as quickly as possible, so structuring the process efficiently helps to improve the overall user experience.

While it's great to have support for bulk inserts directly in Django's ORM, the ORM does not provide much assistance in terms of managing the bulk insertion process itself. One common pattern we found ourselves using for bulk insertions was to:

    build up a list of objects
    when the list got to a certain size, call bulk_create()
    make sure any objects remaining (i.e., which might be fewer than the chunk size of prior calls to bulk_create()) are inserted as well

Since for this particular project we needed to repeat the same logic for a number of different models in a number of different places, it made sense to abstract that into a single class to handle all of our bulk insertions. The API we were looking for was relatively straightforward:

    Set bulk_mgr = BulkCreateManager(chunk_size=100) to create an instance of our bulk insertion helper with a specific chunk size (the number of objects that should be inserted in a single query)
    Call bulk_mgr.add(unsaved_model_object) for each model instance we needed to insert. The underlying logic should determine if/when a "chunk" of objects should be created and does so, without the need for the surrounding code to know what's happening. Additionally, it should handle objects from any model class transparently, without the need for the calling code to maintain separate object lists for each model.
    Call bulk_mgr.done() after adding all the model objects, to insert any objects that may have been queued for insertion but not yet inserted.

Without further ado, here's a copy of the helper class we came up with for this particular project:

from collections import defaultdict
from django.apps import apps


class BulkCreateManager(object):
    """
    This helper class keeps track of ORM objects to be created for multiple
    model classes, and automatically creates those objects with `bulk_create`
    when the number of objects accumulated for a given model class exceeds
    `chunk_size`.
    Upon completion of the loop that's `add()`ing objects, the developer must
    call `done()` to ensure the final set of objects is created for all models.
    """

    def __init__(self, chunk_size=100):
        self._create_queues = defaultdict(list)
        self.chunk_size = chunk_size

    def _commit(self, model_class):
        model_key = model_class._meta.label
        model_class.objects.bulk_create(self._create_queues[model_key])
        self._create_queues[model_key] = []

    def add(self, obj):
        """
        Add an object to the queue to be created, and call bulk_create if we
        have enough objs.
        """
        model_class = type(obj)
        model_key = model_class._meta.label
        self._create_queues[model_key].append(obj)
        if len(self._create_queues[model_key]) >= self.chunk_size:
            self._commit(model_class)

    def done(self):
        """
        Always call this upon completion to make sure the final partial chunk
        is saved.
        """
        for model_name, objs in self._create_queues.items():
            if len(objs) > 0:
                self._commit(apps.get_model(model_name))

You can then use this class like so:

import csv

with open('/path/to/file', 'rb') as csv_file:
    bulk_mgr = BulkCreateManager(chunk_size=20)
    for row in csv.reader(csv_file):
        bulk_mgr.add(MyModel(attr1=row['attr1'], attr2=row['attr2']))
    bulk_mgr.done()

I tried to simplify the code here as much as possible for the purposes of this example, but you can obviously expand this as needed to handle multiple model classes and more complex business logic. You could also potentially put bulk_mgr.done() in its own finally: or except ExceptionType: block, however, you should be careful not to write to the database again if the original exception is database-related.

Another useful pattern might be to design this as a context manager in Python. We haven't tried that yet, but you might want to.

Good luck with speeding up your Django model inserts, and feel free to post below with any questions or comments!
