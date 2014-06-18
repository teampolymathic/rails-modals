# Rails::Modals

Turn your links to Rails forms into modals!

## Installation

Add this line to your application's Gemfile:

    gem 'rails-modals'

Then add the following requires to your `application.js` and `application.css` files:

#### application.js (or similar)

    //= require rails-modals

#### application.css (or similar)

    /*= require rails-modals

## Usage

Add the following to the end of your layouts (this example is for ERB):

    <%= modals if modals? %>

Change all your links that you want to link to a modal to the following:

    link_to ... => link_to_modal ...

That's it! Provided your linked-to pages have forms on them, your forms will populate inside a modal.

Incidentally, the title of the modal window is the same as the `<title>` of your linked-to page.

Options for link_to_modal include:

- `data-precache` => precache the modal (starts loading on page load instead of on user click)
- `data-display-loading` => when the modal is being loaded, show temporary warning text.

## Optimizing your controllers

The forms are requested over XHR, which must follow all redirects. So if you're in the habit of creating controllers that do this:

```ruby
class MyController < ApplicationController
  def edit
    unless @user.pro?
      flash[:alert] = "You have to be upgraded to edit."
      redirect_to root_path
    end
  end
end
```

Then you want to add a case for `request.xhr?`:

```ruby
class MyController < ApplicationController
  def edit
    unless @user.pro?
      if request.xhr?
        render :status => :forbidden, :json => { error: "You have to be upgraded to edit." }
      else
        flash[:alert] = "You have to be upgraded to edit."
        redirect_to root_path
      end
    end
  end
end
```

The response in this case is expected to be a non-`200` error code, and have an `error` key on a root object as shown. If you follow those simple rules, your error message will display in a modal automatically.


## Steps

Sometimes you want to divide a form into steps, like in a wizard.

This is easy: just do the following in your form:

```html
<form action="..." method="POST">
  <fieldset data-modal-step>
    <label>Fields on page 1 of wizard</label>
  </fieldset>

  <fieldset data-modal-step>
    <label>Fields on page 2 of wizard</label>
  </fieldset>
</form>
```

## Remote forms

You can make a modal form remote with

```ruby
link_to_modal "My Link", my_path, remote: true
```

This will make the modal submit remotely - and if you use standard errors like the ones generated by Devise, SimpleForm, etc, you'll even see errors out of the box. (It looks like this.)

```html
<div id="error_explanation">
  <h2>2 errors prohibited this item from being saved:</h2>
  <ul><li>My message</li></ul>
</div>
```

In your controllers, we suggest you append `unless request.xhr?` when you set a flash message, though, to avoid confusing flash messages the next time the user loads a page after an errored modal.

```ruby
def create
  unless @item.save
    flash[:alert] = "There was an error saving your item." unless request.xhr? # <-- like this
  end
end
```

You can also simply return JSON of the format:

```json
// a string:
{ "error": "My error message" }

// or an array of messages
{ "error": ["My error message 1", "My error message 2"] }
```

## Events

`modal:show` when the modal first opens (modal HTML may not be loaded)
`modal:page` when a new modal page is displayed (HTML is loaded)

## Contributing

1. Fork it ( http://github.com/<my-github-username>/rails-modals/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
