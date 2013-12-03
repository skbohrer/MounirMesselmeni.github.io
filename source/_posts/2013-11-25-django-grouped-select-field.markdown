---
layout: post
title: "Django grouped select field for ModelChoiceField or ModelMultipleChoiceField"
date: 2013-11-25 11:08
comments: true
categories: [Django, Python, Custom Fields]
keywords: django, select, grouped, custom fields, ModelChoiceField, ModelMultipleChoiceField, grouped select, groupe choices, choices, field
description: create django grouped select field for ModelChoiceField or ModelMultipleChoiceField
---

Django forms's builtin fields like ChoiceField, ModelChoiceField use by default `forms.Select` widget, but what we do if we need a Select with grouped options.

I found this awesome [snippets][1] that's helped me to achieve this, just in my case I needed to add a line of code to show only options with parent.

<!-- more -->
## Example

Here's a model example:

{% codeblock models.py lang:python %}

class Category(models.Model):

    name = models.CharField(max_length=100)
    parent = models.ForeignKey('self', null=True, blank=True)

    def __unicode__(self):
        return u'{0}'.format(self.name)

{% endcodeblock %}

I created a form that use this model:

{% codeblock forms.py lang:python %}

class ExampleForm(forms.Form):

	category = GroupedModelChoiceField(
        label=_('Category'),
        group_by_field='parent',
        queryset=Category.objects.all(),
    )

{% endcodeblock %}

The snippet show categories with None label, I don't need to make user able to select Categories, only childs can be selected. So, I just edited the line 50 of the snippet to remove group categories.

{% codeblock fields.py lang:python %}

from itertools import groupby

from django.forms.models import ModelChoiceIterator, ModelChoiceField


class GroupedModelChoiceField(ModelChoiceField):

    def __init__(self, group_by_field, group_label=None, *args, **kwargs):
        """
        group_by_field is the name of a field on the model
        group_label is a function to return a label for each choice group
        """
        super(GroupedModelChoiceField, self).__init__(*args, **kwargs)
        self.group_by_field = group_by_field
        if group_label is None:
            self.group_label = lambda group: group
        else:
            self.group_label = group_label

    def _get_choices(self):
        """
        Exactly as per ModelChoiceField except returns new iterator class
        """
        if hasattr(self, '_choices'):
            return self._choices
        return GroupedModelChoiceIterator(self)
    choices = property(_get_choices, ModelChoiceField._set_choices)


class GroupedModelChoiceIterator(ModelChoiceIterator):

    def __iter__(self):
        if self.field.empty_label is not None:
            yield (u"", self.field.empty_label)
        if self.field.cache_choices:
            if self.field.choice_cache is None:
                self.field.choice_cache = [
                    (self.field.group_label(group), [
                     self.choice(ch) for ch in choices])
                    for group, choices in groupby(
                        self.queryset.all(),
                        key=lambda row: getattr(row, self.field.group_by_field))
                ]
            for choice in self.field.choice_cache:
                yield choice
        else:
            for group, choices in groupby(
                    self.queryset.all(),
                    key=lambda row: getattr(
                        row, self.field.group_by_field)):
                if group is not None: #Line added
                    yield (
                        self.field.group_label(group),
                        [self.choice(ch) for ch in choices])


{% endcodeblock %}

Okay, now this custom field work, but what if I need a grouped select multiple with `ModelMultipleChoiceField` ?

We can just add a new field like `GroupedModelChoiceField` that inherits from `ModelMultipleChoiceField`

{% codeblock fields.py lang:python %}

from django.forms.models import ModelMultipleChoiceField


class GroupedMultipleModelChoiceField(ModelMultipleChoiceField):

    def __init__(self, group_by_field, group_label=None, *args, **kwargs):
        """
        group_by_field is the name of a field on the model
        group_label is a function to return a label for each choice group
        """
        super(GroupedMultipleModelChoiceField, self).__init__(*args, **kwargs)
        self.group_by_field = group_by_field
        if group_label is None:
            self.group_label = lambda group: group
        else:
            self.group_label = group_label

    def _get_choices(self):
        """
        Exactly as per ModelChoiceField except returns new iterator class
        """
        if hasattr(self, '_choices'):
            return self._choices
        return GroupedModelChoiceIterator(self)
    choices = property(_get_choices, ModelMultipleChoiceField._set_choices)


{% endcodeblock %}

[1]: https://djangosnippets.org/snippets/1968/
