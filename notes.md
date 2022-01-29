# Django:
Getting started: https://docs.djangoproject.com/en/4.0/intro/tutorial01/ - highly recommend part 1 + part 2 of this tutorial just to familiarize with Django first, but the View, Model, and Serializer are going to be slightly different than what they provide. This tutorial also doesn't use a `routes.py` file unlike the template

* Run `python manage.py startapp <app name>` 
* Files you'll need (a few of these are created by default, some you need to make yourself): `<app name>/views.py`, `<app name>/models.py`, `<app name>/routers.py`, `<app name>/serializers/<serializer name>.py`, `core/urls.py`, `core/settings.py`

## Create a Model:
* In `models.py`, use this template:
```
from django.db import models
class YourClass(models.Model):
    db_item1 = models.CharField(max_length=200) # Field 1
    db_item2 = models.TextField() # Field 2
    db_item3 = models.DateTimeField() # Field 3
    # ... etc, more fields
    def __str__(self):
        return # your string here
```
List of fields can be found here: https://docs.djangoproject.com/en/4.0/ref/models/fields/ make sure it corresponds to the appropriate db field (ex. study_name = varchar = CharField, date = datetime = DateTimeField)

## Create a View: 

* Open `<app name>/views.py`  to create a view (view = Python function that takes a web request and returns a response)
* For a VERY basic function that just saves into the DB, you can use this template:
```
class YourViewSet(viewsets.ModelViewSet):
    http_method_names = ["post"]
    permission_classes = (AllowAny,)
    serializer_class = YourSerializerHere 

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save() # Saves to DB

        return Response(
            {
                "success": True,
                "msg": "Success Message",
            },
            status=status.HTTP_201_CREATED,
        )
```
* We haven't created a serializer yet, but from what I understand it does is essentially reads the request data and parses it in a way that's usable in Python

## Create a Serializer:
* Create a file+folder `<app name>/serializers>/your_serializer.py` 
* Similar to the model, write out your data fields
* Create a `Meta` class that uses the Model you wrote earlier and fields corresponding to the Model you wrote earlier
```
class StudiesSerializer(serializers.ModelSerializer):
    field1 = serializers.CharField(min_length=4, max_length=128, write_only=True)
    field2 = serializers.CharField(required=True)
    field3 = serializers.DateField(required=True)

    class Meta:
        model = YourModel
        fields = ["field1", "field2", "field3"]
```
* There are apparently lots of ways to write a Serializer but this is the method they use in the template so I followed it
  
## Setting up the URL:
* Open `core/urls.py`
* In the `urlpatterns`, add another pattern: `path('api/<base_url>/', include('appname.routers')),`
* Your base url can be anything - ex. If I wanted to create a url for studies, I would do `path('api/studies/', include('studies.routers'))`
* Open/create `<app name>/routers.py`
* Code:
```
from rest_framework import routers
from appname.views import YourViewSet

router = routers.SimpleRouter(trailing_slash=False)

router.register(r"your-endpoint", YourViewSet, basename="base_url")

urlpatterns = [
    *router.urls,
]
```
* `router.register` registers your endpoint to the basename provided, ex. if I used `router.register(r"create-study", YourViewSet, basename="studies")` then my endpoint will be at `localhost:5000/api/studies/create-study`
DB Setup: https://docs.djangoproject.com/en/4.0/intro/tutorial02/
* In `core/settings.py`, add your new app to the `INSTALLED_APPS` section
```
INSTALLED_APPS = [
    'appname.apps.AppNameConfig',
...]
```
* Run `python manage.py makemigrations appname` - you should see something printed about your Models being created
* Run `python manage.py migrate` to create model tables in the DB
DB is now set up! (Note: you do not need to manually create any tables yourself - `migrate` automatically does it for you based on the Model you created)

At this point, you should manually check your new endpoint - I recommend using Postman or something similar to try sending a POST request and see if it shows up in the db

# Connecting to React: 
* In `src/api`, create a new file with the following (note: basically copied from template)
```
import axios from "./index";

class YourApi {
  static yourFunction = (data) => {
    return axios.post(`${base}/your-endpoint`, data);
  };
}

let base = "base_url";

export default YourApi;
```
* Now in your component, you can call the function like so: 
```let response = await YourApi.yourFunction({
        field1: field1_data,
        field2: field2_data,
        field3: field3_data,
      });
```
* How you decide to call the function is up to you - ex. as a Button onclick, when loading a page, etc.

# Final Notes
You can see my (experimental) branches here - warning that the code is terrible atm and I won't be merging any of this into main, but might be useful as a reference in the meantime
* https://github.com/ibrahimq1/ro_frontend/commit/97e8aa129eeb10383de0d928d41dbe36f9cf13f1
* https://github.com/ibrahimq1/ro_backend/commit/2198cdd82df108827dd15a888d0bc42d203eb11c