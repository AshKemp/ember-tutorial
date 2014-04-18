# Editing a Lead Part II

Now we're going to do the other half of editing. A user clicks an edit link and sees fields to edit the lead's name, phone, and email.

This UI will replace the UI that shows the lead, so we'll have to do some special work to handle that.

## Add a Route

Here we go. As always, add a route first. We will place an `edit` route under our existing `lead` resource. The `lead` resource handles fetching the lead. We shouldn't repeat that work if we don't need to.

```coffee
# app/assets/javascripts/router.js.coffee
@resource 'lead', path: 'leads/:id', ->
  @route 'edit'
```

Note that you need to add a `, ->` after the `lead` resource, othwerise you'll get an error.

This route is going to look for a `LeadEdit` controller, view, and template.

## Create the Template

I'm going to add the template first this time because it will inform us about what actions we need to handle in the controller.

This template must be located in `templates/lead/`. Since the route is nested inside a `resource`, Ember expects the template to be inside a subdirectory with the name of the resource. If you're not sure where to place a template just look in the Ember Inspector's "Routes" tab to find out.

```
# app/assets/javascripts/templates/lead/edit.js.emblem

article#lead
  h1
    fullName

  form
    fieldset
      dl
        dt: label for=view.firstName.elementId First Name:
        dd: view Ember.TextField value=firstName viewName="firstName"

      dl
        dt: label for=view.lastName.elementId Last Name:
        dd: view Ember.TextField value=lastName viewName="lastName"

      dl
        dt: label for=view.email.elementId Email:
        dd: view Ember.TextField value=email viewName="email"

      dl
        dt: label for=view.phone.elementId Phone:
        dd: view Ember.TextField value=phone viewName="phone"

    fieldset.actions
      input type='submit' value='Save Changes' click="saveChanges"
      a.cancel href='#' click="cancel" cancel
```

Ember provides the `Ember.TextField` view which renders a text input. Just assign `value` to the property you want to bind to. For example, for `firstName` text field you could assign the value to `model.firstName` or just `firstName`. The template always looks in the controller first for a property, and if it doesn't find it then it will look to the model.

The `form` tag actually doesn't do anything, it's just there to provide proper markup.

This template has two actions: `saveChanges` and `cancel`. Let's implement them now.

## Create the Controller

```coffee
# app/assets/javascripts/controllers/lead_edit.js.coffee
App.LeadEditController = Ember.ObjectController.extend

  actions:

    saveChanges: ->
      @get('model').save().then =>
        @transitionToRoute 'lead'

    cancel: ->
      @get('model').rollback()
      @transitionToRoute 'lead'
```

This part is fairly simple, though you might not be familiar with `.then`. `save()` returns a `Promise` object, which we can call `.then` on to execute code when the promise is resolved. Basically what this means is that `transitionToRoute` won't be called until the server has confirmed that the model was saved.

## Add the Edit Link

We need a way to get to our new route. Add a link inside the `h1` tag in the `lead` template. Don't worry, the stylesheet will make it look pretty.

```coffee
# app/assets/javascripts/templates/lead.js.coffee
h1
  fullName
  link-to 'edit' 'lead.edit' model classNames='edit'
```

## Try It

Open the browser and try clicking the edit link. You should see the URL change, but nothing else should happen. Can you figure out why?

**Outlets**, don't forget about them. If a template doesn't appear, always ask yourself: **did I add an outlet???** Since our `edit` route is nested under `lead`, Ember is trying to render the template into an `outlet` tag inside the `lead` template. Add it to the top of the `lead` template:

```coffee
# app/assets/javascripts/templates/lead.js.emblem
outlet
```
Now try it. It should work, but now we have a new problem: the show UI for a lead is still be present. That's because **nested routes means nested UI**. Since the `lead` resource is still active, the UI is still active.

There's a fairly simple fix to this. We'll simply hide the show UI when we're editing.

## I Heard You Like Editing

We'll create an `isEditing` property on the `lead` controller. If it's true, we'll hide the UI we don't want to see.

First add `unless isEditing` to the template and indent all the show UI under it:

```
# app/assets/javascripts/templates/lead.js.coffee
outlet

unless isEditing
  article#lead
    h1
      fullName
      link-to 'edit' 'lead.edit' model classNames='edit'
  # etc... 
```

Now whenever we visit the edit route we need to set `isEditing` to true. We can do that inside a `LeadEdit` route. We haven't made one yet, so do it now:

```coffee
# app/assets/javascripts/routes/lead_edit.js.coffee
App.LeadEditRoute = Ember.Route.extend

  activate:   -> @controllerFor('lead').set 'isEditing', true
  deactivate: -> @controllerFor('lead').set 'isEditing', false
```

Now we see those route hooks coming in handy! On `activate` we get the `LeadController` and simply set `isEditing` to true. On `deactivate` we do the opposite. And boom, we're done.

We could do one last thing for clarity: we could add `isEditing` to the `LeadController`.

```coffee
# app/assets/javascripts/controllers/lead.js.coffee
App.LeadController = Ember.ObjectController.extend

  isEditing: false

  #etc...
```

This way future programmers in our code will know that we have an `isEditing` property and it should be `false` by default. We don't have to do this but I think it's good style.
