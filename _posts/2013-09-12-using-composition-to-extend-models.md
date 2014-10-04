---
title: Adding functionalities to models using composition
layout: basic
category: models
---

Adding functionalities to models using composition
--------------------------------------------------


###problem:
You want to add a complex capability to a [Django model](https://docs.djangoproject.com/en/1.5/topics/db/models/),
your are tempted to write a mixin or a subclass,
but you have heard that one should "favor composition over inheritance".
How could you apply this concept to adding funcitonality to Django models?


###solution:
Use a **"field-like" object** which can be added and removed at will from any application model classes.
Such an object will appear in the model class definition just like a standard model fields, although it will not deal with db persistence.
The object is 'field-like' because it implements a 'contribute_to_class' method, allowing it to plug into Django's metaprogramming just like other model fields.
(one can see a form of [duck typing](http://en.wikipedia.org/wiki/Duck_typing) in this).

Any model class can then delegate complex functionalities log to such a field-like object. Models with various
new features can be put together through composition, which is similar to what developpers are already doing
when composing their application models using various Django model fields.

A field-like object will usually consist of two parts: an object that implements `contribute_to_class` and who is added to the
model class definition in the app's models.py. This object is then responsible for the necessary configuration during Django's metaprogramming.
Usually this object will subsequently replace itself on the model class with a descriptor to manage access to the attribute by application code.
Those two steps can be combined in one class.

If you only need to manage access to your class attribute, it might be easier to just use a descriptor directly without doing anything with
`contribute_to_class`. That method is however very useful if you need to configure anything prior to the attribute being accessed.
Via this method Django gives you a change to intercede while te model class is being prepared during Django's metaprogramming.

###discussion:
Python gives you an excellent tool to use composition and delegation in the form of [descriptors](http://docs.python.org/3.3/howto/descriptor.html).
These can be added to any class definition just like a regular attribute, when the attribute is accessed by application code,
the descriptor's `__get__` method is called with a reference to the class it was attached to as well as the instance
whose attribute is being accessed. This gives the descriptor the opportunity to configure and return any type of value.
In this case the descriptor could return an object that represents the complex functionality added to the model. 

Django itself gives you an additional hook to start the configuration of your field-like object even earlier.
If your obejct implements a `contribute_to_class` method, Django will call this method when the BaseModel metaclass is
processing the model class to which the object was attached. This allows you to participate in Django's own metaprogramming efforts.
This can be useful in situations where configuration is needed prior to the attribute being actually accessed on an instance.
For example, this is a good opportunity to register signal handlers, ensuring registration is done before any application code runs.

These tools allows you to write independent classes that are firmly focused on providing a solution to one particular problem and which can be effectively tested in isolation.
Since all logic is hidden away in this new class, you can hide all implementation detail from the application using your object, making it easy for
others to use your code and retaining full flexibility on your side to change the internals of your object in later releases.
You will also be forced to design a clean API and comminucate it effectively to users of your code. This will greatly improve the quality of your code.
Your field-like objects can also be easily re-used in other apps and projects. They are also perfect candidates for addition to the [cheese shop](https://pypi.python.org/pypi). 

###example:
A field-like object that would automatically update geolocation data on any model each time the model is saved.
This object is configured with the necessary names of fields where the address details can be found. This simple attributes
can be added to any model without the need for inheritance. The contribute_to_class method is used to register pre_save signal
before any application code is run. This is a good example of it's usefulness as a normal descriptors wouldn't be
able to timely register pre_save handlers as it's code only attributes once the attribute is accessed. 

{% highlight python %}
from django.db.models.signals import pre_save, post_save, m2m_changed
from geopy import distance, geocoders

class GeoInfo(object):
    '''just a container of data, gets returned to application code accessing the attribute on a model'''
    
    def __init__(self, latitude=None, longitude=None):
        self.latitude = latitude
        self.longitude = longitude
    
class PreSaveGeoCoder(object):
    '''the inner logic, remains fully hidden from application code using your object
    allowing you to change these internals anytime'''
    
    def __init__(self, model_cls=None, address=None):
        self.model = model_cls
        self.address = address
        self.geo_info = GeoInfo()
        
    def __call__(self, sender, instance, *args, **kwargs):
        self.sender = sender
        self.instance = instance
        self.set_geo()
		return self.geo_info
        
    def set_geo(self):
	'''do some geo-coding here'''
        self.geo_info.latitude = lat 
        self.geo_info.longitude = longitude
            
class GeoManager(object):
    '''the field-like object that is added as an attribute of application models
    Note this is one combining a descriptor and also implementing a contribute_to_class method to
    plug into Django's metaprogramming'''
    
    def __init__(self, street_field=None, number_field=None, postal_code_field=None, city_field=None):
	'''get the arguments when the descriptor is attached to the class and initialized'''
        self.address_info = dict(street=street_field,
                            number=number_field,
                            postal_code=postal_code_field,
                            city=city_field)
    
    def contribute_to_class(self, cls, name):
	'''Django metaprogramming hook, called by Django when the class to which this obejct is attached is processed by ModelBase'''
        self.geo_coder = PreSaveGeoCoder(model_cls=cls, address=self.address_info)
        pre_save.connect(self.geo_coder, sender=cls, weak=False)
        setattr(cls, name, self)

    def __get__(self, instance, owner):
	'''The method called by Python when the attribute is accessed by code'''
        if not instance:
            return
        return self.geo_coder()


'''example use in a model'''
class Place(models.Model):
    name = models.CharField(max_length=200, null=True, blank=True)
    street = models.CharField(max_length=200,null=True, blank=True)
    number = models.CharField(max_length=50, null=True, blank=True)
    postal_code = models.CharField(max_length=50, null=True, blank=True)
    city = models.CharField(max_length=200, default='Paris', null=True, blank=True)
    
    geo_info = GeoManager(street_field='street',
            number_field='number',
            postal_code_field='postal_code',
            city_field='city')

'''elsehere in your project you can do'''
place = Place.objects.get(name='Ye old Sheshire')
place.geo_info.latitude 
place.geo_info.longitude
{% endhighlight %}

###How Django itself uses this technique:
A good example of this technique is Django's own **django.db.models.manager.Manager** class. A manager instance is set directly set on a model class,
the ModelBase metaclass then calls the manager"s contribute_to_class method, allowing the manager to
configure itself as well as the class it was attached to. The manager then replaces itself on the model class with a descriptor
whose job is to check whether the attribute is only accessed at the model Class level. When the value is accessed on a model class, the descriptor returns the manager.
If the attribute is accessed on an instance, the descriptor raises an error.

{% highlight python %}
class Manager(object):
    # Tracks each time a Manager instance is created. Used to retain order.
    creation_counter = 0

    def __init__(self):
        super(Manager, self).__init__()
        self._set_creation_counter()
        self.model = None
        self._inherited = False
        self._db = None

    def contribute_to_class(self, model, name):
        # TODO: Use weakref because of possible memory leak / circular reference.
        self.model = model
        # Only contribute the manager if the model is concrete
        if model._meta.abstract:
            setattr(model, name, AbstractManagerDescriptor(model))
        elif model._meta.swapped:
            setattr(model, name, SwappedManagerDescriptor(model))
        else:
        # if not model._meta.abstract and not model._meta.swapped:
            setattr(model, name, ManagerDescriptor(self))
        if not getattr(model, '_default_manager', None) or self.creation_counter < model._default_manager.creation_counter:
            model._default_manager = self
        if model._meta.abstract or (self._inherited and not self.model._meta.proxy):
            model._meta.abstract_managers.append((self.creation_counter, name,
                    self))
        else:
            model._meta.concrete_managers.append((self.creation_counter, name,
                self))
                
    '''a lot more stuff goes on down here, have a look at django.db.models.Manager'''
                
class ManagerDescriptor(object):
    # This class ensures managers aren't accessible via model instances.
    # For example, Poll.objects works, but poll_obj.objects raises AttributeError.
    def __init__(self, manager):
        self.manager = manager

    def __get__(self, instance, type=None):
        if instance != None:
            raise AttributeError("Manager isn't accessible via %s instances" % type.__name__)
        return self.manager

'''This is what you are using when you do:'''
MyModel.objects.all()
{% endhighlight %}

As this example shows, using contribute_to_class allows you to plug into the metaprogramming
that Django performs when your application models are processed by Django.
This is a great way to configure the model class to which your object was attached to.
After this is done, you will typically use the "name" argument to set an appropriate
value as atttribute of the fully prepared model class. In most cases a setting a descriptor on the class will give
you a lot of flexibility to chose what value to return when the attribute is accessed by application code.

If you do not require the actual name used to add the attribute to the model class and you do not require access to the class prior to
attribute access for further configuration, you can probably skip the first step of using an object that implements the
contribute_to_class method and rather directly set a descriptor to the class which will still give you the flexibility you need to
configure and return a complex object at attribute access time.


####More info on composition in Python:
* [Inheritance vs. Composition](http://learnpythonthehardway.org/book/ex44.html)

####More info on Django field-like objects:
* An excellent source of advanced Django recipes is the book [ProDjango](http://prodjango.com/) by [Marty Alchin](http://martyalchin.com/)
  Chapter 11 of the 2008 edition has an excellent example of this technique in the form of the [HistoryRecords](https://github.com/smn/django-historicalrecords) field-like object.