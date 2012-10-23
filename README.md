## SupportBee App Platform

### Introduction

SupportBee Apps are the easiest, yet powerful way of extending/customizing SupportBee helpdesk. This platform exposes the API in a easily consumable way. The Apps are hosted on SupportBee servers.

If you are new to SupportBee, please have a look at how it works [here](https://supportbee.com)

### How does the App Platform Work?

An **App** is deeply integrated with SupportBee helpdesk. It can receive **Events** from SupportBee. It can also define **Actions**.  

_Events_ are triggered by SupportBee during various times of the lifecycle of a ticket. Currently the platform supports the following _Events_:
* Ticket Created
* Agent Reply Created
* Customer Reply Created
* Comment Created
An App can consume one, many or all events. For example an App can send an SMS to a cell when the event "Ticket Created" is triggered.

_Actions_ are triggered by the user of SupportBee helpdesk from the User Interface. Currently the platform supports a single action called _Button_. If an App defines a Button action, a UI component is rendered for Ticket Listings in the SupportBee UI as shown below

![The Button]()

We will go more into this later.

### Writing an App

An App resides in the ``/apps`` folder of the App Platform. Each app has the following structure:

```
dummy
|--assets
|  |--views
|     |
|
|--dummy.rb
|--config.yml
```

#### config.yml
Each app has a ``config.yml`` where all the configurations of the app are defined. 

```
name: Dummy
slug: dummy
access: public
description: This is to test and a boilerplate app.
developer: 
  name: SupportBee
  email: SupportBee
  twitter: @supportbee
  github: SupportBee
action:
  button:
    screens: 
    - all
    - unassigned
    listing: 
    - all
    label: Send To Dummy
```

The ``slug`` should be unique across the platform.


#### {slug}.rb
The app logic is defined in this file. The whole app can be defined in this single file or can be spread across multiple files which are required here. The basic structure is as follows:

```
module Dummy
  module EventHandler
    def ticket_created; end
    def ticket_updated; end

    def reply_created; end
    def reply_updated; end

    def all_events; end
  end
end

module Dummy
  module ActionHandler
    def action_button
     # Handle Action here
    end

    def all_actions
    end
  end
end

module Dummy
  class Base < SupportBeeApp::Base
    string :name, :required => true
    password :key, :required => true, :label => 'Token'
    boolean :active, :default => true
  end
end
```

#### Define Settings
An app can specify the settings required by it in the ``Base`` class. These settings are accepted when the app is added to a SupportBee helpdesk. 

```
module Dummy
  class Base < SupportBeeApp::Base
    string :name, :required => true
    password :key, :required => true, :label => 'Token'
    boolean :active, :default => true
  end
end
```

An app can define a ``string``, ``password`` or a ``boolean`` type of setting. Each setting accepts a ``name`` of the settings and a set of options

* :label; if not defined, the name is humanized and rendered as the label
* :required
* :default

![The Setting]()

#### Consume Events
An App can consume events by defining methods in the ``EventHandler`` module.

```
module Dummy
  module EventHandler
    def ticket_created; end
    def ticket_updated; end

    def reply_created; end
    def reply_updated; end

    def all_events; end
  end
end
```

The event ``ticket.created`` triggers the method ``ticket_created`` and so on. The method ``all_events`` if defined is triggered for all events.

All the methods have access to the following information:
* **auth**: This is required to get SupportBee API access for the helpdesk which triggered the App. 
* **settings**: This contains the values of the settings defined by the app for the helpdesk which triggered the App.
* **payload**: This contains the event/action relavent data. This changes depending on the type of event or action.

Here is an example of a Campfire App posting to campfire on ticket creation.

```
def ticket_created
  campfire = Tinder::Campfire.new settings.subdomain, :token => settings.token
  room = campfire.find_room_by_name(settings.room)
  room.speak "New Ticket: #{payload.ticket.subject}"
end
```

#### Respond To Actions
