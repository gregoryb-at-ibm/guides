# Guide for adding a "new" button 
You need to you already have an application record and a GUI controller that implements a GUI list view (including templates)
This guide will tell you how to add a button to the list view that creates new instances of your application record.\
In this guide we will implement the actual creation of the record in an EMS provider. It could be adjusted to implement it in manageiq core if it doesn't require interaction with an EMS.

The guide assumes that 
* Your application record is in `app/models/your_item.rb`
* Your application-record belongs to an EMS. Usually its instances appear in MIQ by being refreshed from the EMS.
* Your ui controller is in `plugins/manageiq-ui-classic/app/controllers/your_item_controller.rb`
* Your provider is in `plugins/manageiq-providers-your-provider`
* A menu for your item's list view `plugins/manageiq-ui-classic/app/heplers/application_helpers/toolbar/your_items_center.rb`

You will need to make changes to the following repositories:
* manageiq
* manageiq-ui-classic 
* manageiq-api
* manageiq-providers-your-provider (if you implement the creation in an EMS)

## EMS application-record

Make sure you have an application-record in `plugins/manageiq-providers-your-provider/app/models/manageiq/providers/your-provider/your_item.rb`.
It should extend `app/models/your_item.rb`. 

In the application-record in your provider, you need to add two items.

1. `supports :create`
2. `def self.raw_create_your_item(...)` 

The method `self.raw_create_your_item` is where you write the code to sends the request to your EMS to create the actual item.\ 
Here you wll check that the creation went well, and maybe schedule an EMS refresh or reflect the change in some other way in MIQ.

This is assuming that the base application-record in `app/models/your_item.rb` already has `self.create_your_item`, `self.raw_create_your_item`, and `self.create_your_item_queue`.\
If not, add these methods to the base record by copying them from some other model in MIQ.


## Adding a feature for the new operation 
1. In `app/models/ext_management_system.rb` 
   1. add `supports_attribute :supports_create_your_item, :child_model => "YourItem"`
1. In `manageiq-providers-autosde/app/models/manageiq/providers/your-provider/your_manager.rb`, add `supports :add_your_item`

## Toolbar Button
#### Add button to the menu
Find the menu file in `plugins/manageiq-ui-classic/app/heplers/application_helpers/toolbar/your_items_center.rb`.\
Add a button into the menu for creating your items. The button needs to be called `:your_item_new`. You can copy a `..._new` button from another menu and adjust to fit your item.

#### Add button class
Create a button class to handle "new" operation. 
It needs to be in `manageiq-ui-classic/app/helpers/application_helper/button/your_item_new.rb`. You can copy another button and adjust it.

## JS Form
You need to create a form-folder in `manageiq-ui-classic/app/javascript/components/your-item-form/`. With `index.jsx` and `your-item-form.schema.js`.
You can copy from another form and adjust as needed.

Here you determine the GUI form the user will see when creating a new instance of your item. I won't go into the details of it in this guide.\
When developing, it's safest to start with a minial form just to see that it's working.

After you add your form, you need to register it in `component-definitions-common.js`.

## The UI Controller
In your controller in `plugins/manageiq-ui-classic/app/controllers/your_item_controller.rb` you need to add the `new` method.\
You can copy it from another controller and adjust as needed.

Another thing you need in your controller is a method called `specific_buttons(pressed)`. If you don't have it already, copy it from another controller.\
The method consists of a `case`.\
Add the following `when` to the case. If you have just copied the method from another controller you can remove all other `when`s.
```ruby
when 'your_item_new'
  javascript_redirect(:action => 'new')
end
```

## haml template for 'new' operation
Create `new.html.haml` under `plugins/manageiq-ui-classic/app/views/your_item/`. You can copy it from another item and adjust as needed.
Make sure to point at your JS Form which you defined above `react 'YourItemForm'`

## UI routes
In `manageiq-ui-classic/config/routes.rb`, you need to find the section for your_item. Then you need to make sure there's `new` in the get section, and `button` in the post section.

## API Controller
If there isn't one, create a controller for your item in `manageiq-api/app/controllers/api/your_items_controller.rb`. You can copy one of the existing controllers and adjust to your item.

Make sure that there is a `create_resource` method in the api controller. If not, copy if from some other controller and adjust to fit your item.\
This method is supposed to call the method`create_your_item_queue` on the base application-record.

If you are adding a new controller, you'll need to add routes to in `manageiq-api/api.yml`. Copy from one of the existing routes and adjust.



