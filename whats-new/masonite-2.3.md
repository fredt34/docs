# Masonite 2.3

**NOTE: Masonite is currently in Beta 2 Phase. There will be 1 more Beta (Beta 3) until Masonite 2.3 is finally released. The current release is 2.3.0b2. You can install this with `pip install masonite==2.3.0b2`. Note that the docs are not currently updated.**

Masonite 2.3 brings a lot of code and quality of life improvements. There aren't a lot of major ground breaking changes and depending on how your application is structured, may not require much effort to upgrade.

Below is a list of changes that will be in Masonite 2.3:

**Note that some of these features may not yet be documented in the 2.3 docs. Before final release, all docs will be up to date. This document is simply a running list of the new changes.**

**There will also be update docs on how to upgrade your application from Masonite 2.2 to 2.3 when Masonite 2.3 is fully released.**

## Package Changes

Some Larger changes include all packages for Masonite use SEMVER versioning while Masonite still using ROMVER as it has been since Masonite started. 

Masonite will also not require any packages for you through Masonite requirements and will instead put the requirements in the `requirements.txt` file in new projects. This will allow packages to release new Majors outside of the major upgrades of Masonite. So we can develop new package improvements faster.

## Remove TestSuite Class

The `masonite.testing.TestSuite` class has been completely removed. This was a class that was obsolete and has been replace by the `masonite.testing.TestCase` class. The `TestSuite` class was bascially wrappers around some of the commands and predated the newer testing features.

## Removed SassProvider

Recently there has been a good amount of talks about the limitations of compiling sass with the Python `libsass` provider. Because of these limitations, Masonite 2.3 will remove it completely.

## Added Laravel Mix By Default

A Laravel Mix file will now come with new installed applications created by craft.

## Added Responsable Class

The Responsable class will be able to be inherited by any class and be returned in the controller. If the class has a `get_response()` method then the class will be rendered using that method. The `View` class now acts in this way.

## Dropped Support For Python 3.4

Masonite has always supported the previous 4 Python versions. Since the release of Python 3.8 recently, Masonite will now only support Python 3.5, 3.6, 3.7 and 3.8. When Python 3.9 comes out, Python 3.5 will be dropped.

You can still use previous versions of Masonite if you need a previous supported Python version.

## Completely Rewrote The Authentication

Masonite has moved away from the `Auth` class and towards `Guard` classes. These classes will be responsable for handling everything from getting the user to handling logging in and logging out.

There is also a much better way to register new guards so packages will be able to register their own guards with your app which you can then use or swap on the fly.

## Swapped the `Auth` class for a `Guard` class

This is a very small change but will save a lot of time when upgrading your application. Now anytime you resolve the `Auth` class you will get an instance of a `Guard` class instead. The use of both classes are identical so they should just work as is.

(This swap by be removed in Masonite 2.4+)

## Added A New `AuthenticationProvider`

All the authentication stuff in the previous improvements have been abstracted to their own provider so you will need to a add a new provider to your providers list.

## Auth Scaffolding Now Imports Auth Class

The `Auth` class now contains a new method which returns a list of routes. This cleans up the `web.py` file nicely when scaffolding.

## Removed The Ability For The Container To Hold Modules

The container can not longer have modules bound to it. These should instead be imported.

## Added Several New Assertions

Added a few new assertion methods to help chaining methods and keeping tests short and fast. These include `assertHasHeader` and `assertNotHasHeader`.

## Added Download Class

Added a new download class to make it very easy to force downloads or display files.

This is used like this:

```python
from masonite.response import Download

def show(self):
    return Download('/path/to/file', force=True)
```

Forcing will make the file download and not forcing will display the file in the browser.

## Added Preset Command

You can now run a `craft preset` command which will generate some `.json` files and example templates. There is a `react`, `vue` and `bootstrap` preset currently.

## Changed How Query Strings Are Parsed

Instead of a url like `/?filter[name]=Joe&filter[user]=bob&email=user@email.com`

parsing to:

```python
{
    "filter[name]": "Joe",
    "filter[user]": "Bob",
    "email": "user@email.com"
}
```

It will now parse into:

```python
{
    "email": "user@email.com",
    "filter": {
        "name": "Joe",
        "user": "bob"
    }
}
```

Parsing the query to the original way is no longer possible. This also comes with a `query_parse` helper which you can use to parse a query string the same way Masonite does.
 
## Improved Container Collection

The container has a helpful `collect` method which allows you to collect all the classes in the container with a certain key or an instance of a class like this:

```python
app.collect('*Command')
```

Will collect everything in the container where the binding key ends with `Command`.

You can also collect everything that is a subclass:

```python
from masonite.response import Responsable

app.collect(Responsable)
```

This will collect everything that is a subclass of `Responsable`

This has now been expanded to also include instances of. So it will work for objects now and not just classes.

## Moved The `masonite/` Directory To The `src/masonite` Directory

This is an internal change mostly and completely transparent to all who install Masonite. This allows installing third party packages into Masonite with the same namespace without namespace conflicts.

## Changed The Way The WSGI Server Handles Responses

Masonite now handles the response as bytes. This allows for different classes to handle the response in different ways.

Previously Masonite ultimately converted everything to a string at the end but some things needed to be returned to the WSGI server as bytes (like the new file download feature). So if you need to handle the raw response then you will now expect bytes instead of a string.

# Scheduler

The Scheduler has a few changes.

## New Methods

There are a few new methods on the tasks you can use like `every`, `every_minute`, `every_15_minutes`, `every_30_minutes`, `every_45_minutes`,  `daily`, `hourly`, `weekly`, `monthly`.

These can either be used inside the provider or inside the command to make a more expressive scheduling syntax.

## New Scheduling Helper Class

Providers can now inherit the `CanSchedule` class which gives it access to the new `self.call()` method (which is used to schedule commands) and `self.schedule()` method (which is used to schedule jobs).

These are really just helper methods to help bind things to the container more expressively.

## New Scheduling

You can now schedule jobs or commands in a new expressive way. In addition to setting the schedule interval as attributes on the Task you can now do it directly in the provider:

```python
def register(self):
    self.call('your:command --flag').daily().at('9:00')
    self.call('your:command --flag').every_minute()
    self.schedule(YourJob()).every('3 days')
```

There are several other methods that will be documented on release.

## Namespace Change

We also changed the namespace from `scheduler` to `masonite.scheduler`. So you will need to refactor your imports.