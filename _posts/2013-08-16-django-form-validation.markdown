---
layout: post
title:  "Django Form Validation"
date:   2013-08-16
categories: django
tags: forms
---

On the current project I'm working on, users can create a subject and
subsequent topics within each subject. A topic has to be unique within
each subject, but not unique across the app. So Math as a subject can
have only one Theory topic, but English can have a Theory topic as well.

I have a TopicForm (instance of ModelForm) that has a hidden field
called subject, which stores a reference to a Subject object. The hidden
field is a ModelChoiceField, which is basically how forms render a
ForeignKey field from a model. I needed to pass a Subject object from
the previous view to the TopicCreateView and save it with the TopicForm.
In retrospect, the solution was quite easy, but I took a rather
circuitous path to get to it, so documenting it for future use is
probably a good idea.

In my forms.py, the subject field is first hidden. Normally, to hide a
form field, I add the

{% highlight python %}
widget = forms.HiddenInput
{% endhighlight %}

in as an argument, but with the ModelChoiceField, that just broke the
form. I had to add

{% highlight python %}
queryset=Subject.objects.all()
{% endhighlight %}

and that fixed it. I need to figure out why.

{% highlight python %}
class TopicForm(ModelForm):
    subject = forms.ModelChoiceField(label="",
                        queryset=Subject.objects.all(),
                        widget=forms.HiddenInput)
    class Meta:
        model = Topic
        fields = ['title', 'subject']
{% endhighlight %}

After that I needed to set the value of the subject field, which I did
in the form valid method on the TopicCreateView.

{% highlight python %}
def form_valid(self, form):
    obj = form.save(commit=False)
    current_user = get_user(self.request)
    obj.subject = Subject.objects.get(
                id=self.kwargs['subject'],
                submitter_id=current_user.id)
    obj.save()
    return HttpResponseRedirect(self.get_success_url())
{% endhighlight %}

Now that we have the subject set on the form, when we try and save the
new topic, we need to make sure it doesn't exist within that Subject. I
do that in the form's clean method.

{% highlight python %}
def clean(self):
    cleaned_data = self.cleaned_data
    title = cleaned_data.get('title')
    subject = cleaned_data.get('subject')
    if Topic.objects.filter(title=title,
            subject_id=subject.id).exists():
        raise ValidationError(u'Topic already exists!')
{% endhighlight %}

Finally, in my template, I raise an error if the user tries to input the
same topic, letting them know that the topic already exists.

{% highlight python %}
{% raw %}
{% for field in form.visible_fields %}
    {% if form.errors %}
        <div class="form-group has-error">
            <label class="control-label" for="inputError">Topic already exists!</label>
    {% else %}
        <div class="form-group">
    {% endif %}
            <label for="{{ field.html_name }}" class="col-lg-1 control-label">{{ field.label }}</label>
            <div class="col-lg-4">
                {{ field }}
            </div>
        </div>
{% endfor %}
{% endraw %}
{% endhighlight %}

And that solved the problem.
