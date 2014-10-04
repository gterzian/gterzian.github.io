---
title: Manipulating m2m values of models with signals
layout: basic
category: signals
---


Manipulating m2m values of models using signals
-----------------------------------------------

###problem:
You want to manipulate the m2m values of a model within a post or pre_save signal receiver, yet
your changes get wiped out in the subsequent clearing of the m2m values by Django,
which occur after the model save method has returned.

###solution:
To avoid being caught by the clearing of m2m pitfall, register a handler for the m2m_changed signal
with the m2m intermediary model as sender. If you first need to do something in response to a pre_save or
post_save signal, register the handler to the m2m_changed signal from within your save signal handler.

{% highlight python %}
    def save_handler(sender, instance, *args, **kwargs):
        m2m_changed.connect(m2m_handler, sender=sender.m2m_field.through, weak=False)
 
    def m2m_handler(sender, instance, action, *args, **kwargs):
        if action =='post_clear':
            for sub in AnotherModel.objects.all():
                instance.m2m_field.add(sub)
            
    pre_save.connect(save_handler, sender=YourModel, weak=False)
{% endhighlight %}

###discussion:

The intermediary m2m model can be accessed through:

`model.m2m_field_name.through`

This model is the one who fires m2m_changed signals. In addition to the standard
sender and instance args, the receiver to a m2m_changed signal will also receive a third action arg.
Several m2m_changed signals will be fired, and you should only manipulate the m2m value in
response to a m2m_change signal whose action is post_clear. If you do anything before (in the signal with
a pre_clear action) those changes will be wiped out by the subsequent clear. 

Please also note that the instance arg in a m2m_changed handler does not represent the instance of the intermediary m2m
model, but it represents the instance of the original model whose m2m value you want to update. 

