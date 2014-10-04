---
title: Providing a configurable signal handler
layout: basic
category: signals
---

Providing a configurable signal handler
---------------------------------------

###problem:
You need to configure your signal handler prior to it receiving the signal.

###solution:
There is a functional and an object-oriented approach to solving this problem. If the configuration required is fairly simple and could be captured in a few
function arguments: go the functional way. If you are dealing with a more complex need, where the configuration should
probably be handled with several methods: go the OO way.

####functional solution:
Use function closures to preload values into the function that will receive the signal.

example:

{% highlight python %}
def auto_metro_stops_add(manager_name, stations_model=MetroStation, geo_info='geo_info'):
    '''use to handle m2m_changed signal and automatically add relevant metro stops to a model'''
    def handle_m2m(sender, instance, action, *args, **kwargs):
        #do the actual work, using all variables sent along the signal
        #as well as manager_name, stations_model and geo_info        
    return handle_m2m
{% endhighlight %}

{% highlight python %}
'''this one would be configured as a signal handler like this:'''
m2m_changed.connect(auto_metro_stops_add('metro_stops'), sender=Place.metro_stops.through, weak=False)
{% endhighlight %}

###OO solution:
Provide an object as signal handler, pre-configuring it with the `__init__` method and using the `__call__` method as
the signal handler.

example:

{% highlight python %}
class PreSaveGeoCoder(object):
    
    def __init__(self, model_cls=None, address=None):
        self.model = model_cls
        self.address = address
    
    def __call__(self, sender, instance, *args, **kwargs):
        self.sender = sender
        self.instance = instance
        self.set_geo()
        
    def set_geo(self):
        #do the actual geolocation work
{% endhighlight %}

{% highlight python %}
'''Which would be configured as such:'''
pre_save.connect(PreSaveGeoCoder(model_cls=cls, address=self.address_info), sender=cls, weak=False)
{% endhighlight %}


