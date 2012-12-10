backbone.activities
===================

Backbone Activities is a [Backbone](https://github.com/documentcloud/backbone) plugin which makes it easier to create and organize responsive web apps. It borrows ideas, and its name, from Android's Activities, so some concepts may be familiar to Android developers.

Two additional Backbone entities are provided: `Backbone.Activity` and `Backbone.ActivityRouter`, which extends `Backbone.Router`.

Dependencies are [Backbone](https://github.com/documentcloud/backbone) and [Backbone.layoutmanager](https://github.com/tbranyen/backbone.layoutmanager).


### Activities
In a responsive app, you may have multiple pages for devices with smaller form factors that become a single page on a device with a larger form factor. For example, you might have separate list and detail pages for small devices but a single list/detail page for larger ones. An activity should encompass the behaviour and layouts for a single page on the largest form factor that you support. In the list/detail example, the list and detail pages would be handled by a single activity.

In Backbone, each page for your smallest form factor will have its own route, and so an activity may encompass several routes. In the list/detail example, your activity's routes might be `"!/list"` and `"!/detail/:id"`.

The role of an activity is generally to be responsible for handling all of the data involved in all of its views, and to render a page based on the route that was fired and the current layout by using the `updateRegions` method.

### The activity lifecycle
```
#################################################
# Activity                                      #
#      ################################         #
#      # onCreate()                   #<===========|
#      #  if this is the first time   #         #  |
#      #  time the activity is used   #         #  |
#      ################################         #  |
#                     v                         #  |
#      ################################         #  |
#      # onStart()                    #<===========|
#      #  if the last route was       #         #  |
#      #  handled by another activity #         #  |
#      ################################         #  |
#                     |                         #  |
#  #########################################    #  |
#  # route handler    v                    #    #  |
#  #   ################################    #    #  |
#  #   # onStart(route_params)        #<======| #  |
#  #   #  if the route changed        #    #  | #  |
#  #   ################################    #  | #  |
#  #                  v                    #  | #  |
#  #   ################################    #  | #  |
#  #   # <layout>(route_params)       #<=| #  | #  |
#  #   #  if the route or layout      #  | #  | #  |
#  #   #  changed                     #==| #  | #  |
#  #   ################################    #  | #  |
#  #                  v                    #  | #  |
#  #   ################################    #  | #  |
#  #   # onStop()                     #    #  | #  |
#  #   #  if the route changed        #=======| #  |
#  #   ################################   #     #  |
#  #                  |                   #     #  |
#  ########################################     #  |
#                     v                         #  |
#      ################################         #  |
#      # onStop()                     #         #  |
#      #  if the next route is        #         #  |
#      #  handled by another activity #============|
#      ################################        #
#                                              #
################################################

```

### Hooking up routes to activities; route handlers
An activity's `routes` property defines the routes that an activity is responsible and the keys of the route handler objects for those routes. For example, the following `routes` object designates responsibility for the `"!/list"` route to the activity's `list` route handler:

```
{
  "!/list": "list"
}
```

When the activity router handles the `"!/list"` route, it looks up the activity that claimed that route and then the route handler in that activity by the value for the route in the routes object; in this case, `list`.

A route handler may have `onStart` and `onStop` methods, as well as methods for the application's layouts. Here's an example of an activity with a `list` route handler:

```
var myActivity = Backbone.Activity.extend({
  
  "list": {

    onStart: function() {
      // some setup stuff, perhaps
    },

    onStop: function() {
      // some cleanup stuff, perhaps
    },

    "single": function() {
      // display the single-pane layout

      this.updateRegions({

        "main": {
          template: "my-list-single-template",
          views: {
            ".list-container": new MyListView({ 
              collection: myCollection 
            })
          }
        }

      });

    }

  },

  "routes": {
    "!/list": "list"
  }

});
```

### Regions, and rendering using `updateRegions`
The `updateRegions` method takes an object of views to be inserted into regions. These regions are LayoutManager layouts whose `el`s are typically present throughout the application. It's possible you'll only need a main region, but you might want one for a header, footer, navigation menu, etc. Each activity has access to the app's regions via the `regions` property, which is populated by the activity's activity router so you don't have to worry about them in your activities. I've not yet had reason to access `regions` directly from an activity.

There are a few ways to set the views for a given region using `updateRegions` (or `updateRegion` when updating a single region). The first, and easiest way, is to pass a view, which is inserted directly into the region. The second is to pass an object with a template name and views object. In this case, the region gets the given template rendered into it, and then views are rendered into place in the template using the selector strings. The third is to pass an array of views, but because of the way that LayoutManager works, you should only do this if all of the views in the array use the same template, or the views could render out of order.

### Activity routers and layouts
Activity routers are an extension of `Backbone.Router` with some additional setup logic (to hook up routes to activities and activities to regions), and routing logic (implementing activity lifecycles).

An activity router needs a few things at initialization time:
- `regions`: an object of region names to LayoutManager layouts
- `el`: a DOM element on which to set a class corresponding to the current layout. This lets you hook CSS into layouts
- `activities`: a object of activity names to activities
- `defaultRoute`: an object which defines the activity name and route handler name that should be used for the empty route

The activity router also has a very important method, `setLayout`, which should be called immediately after the router has been instantiated and also whenever the app's layout changes. `setLayout` takes a single string as its argument which is maps to the names of the layout methods of activities' route handlers. You could hook up to a `matchMedia` listener like `enquire.js` to call `setLayout` in order to trigger the layout to change when the app resizes.

### Manual Routing
If you need to programmatically trigger routes, you can use the Activity's `navigate` method. This delegates to `Backbone.history.navigate` if you pass a URL fragment as the first parameter. You can also trigger silent navigation by passing an activity name and handler name, and optionally an arguments object. If you do not want a handler to be reachable via a URL then simply omit it from the routes for the activity.