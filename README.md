# Django REST Framework

- a flexible tool kit for building Restful APIs

# Optimization Notes

## TIP 1: How to increase speed of queries for models with Foreign keys

```python
Object.select_related("field_name")
```

eg. usage: 

### Model
```python
class Task(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.TextField()
    description = models.TextField(blank=True, null=True)
    completed = models.BooleanField(default=False)
    deadline = models.DateTimeField(blank=True, auto_now_add=True)
    created_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)
    owner = models.ForeignKey(settings.AUTH_USER_MODEL, related_name="tasks", on_delete=models.CASCADE)

    class Meta:
        ordering = ["created_date"]


    def save(self, *args, **kwargs):
        super(Task, self).save(*args, **kwargs)

```


### View
```python
TaskModelViewSet(ModleViewSet)

  # Instead of retrieving Task.objects.all()
  # queryset = Task.objects.all()

  # Use the ff:
  queryset = Task.objects.select_related("owner")


```

- The idea of this is to decrease the number of hits in the database resulting to a faster result
- We are instead prefetching the data for the owner field to decrease the number of queries needed in the database
- Note that this is effective for data that have foreign keys

https://docs.djangoproject.com/en/4.0/ref/models/querysets/#select-related


# TIP 2: How to override id return data in api requests

Use slug related field: this will allow you to define what key to use in adding data

```python
class AlbumSerializer(serializers.ModelSerializer):
  tracks = serializers.SlugRelatedField(
    many=True,
    read_only=True,
    slug_field="title"
    )

class Meta:
  model = Album
  fields = ["album_name", "artist", "tracks"]
```

eg. instead of: 

```json
{
  "album_name": "Dear John",
  "artist": "Loney Dear",
  "tracks": [
    23,
    24,
    26,
    ...
  ],
}
```
```json
{
    "album_name": "Dear John",
    "artist": "Loney Dear",
    "tracks": [
        "Airport Surroundings",
        "Everything Turns to You",
        "I Was Only Going Out",
        ...
    ]
}
```

