# How to MOVE from MVC

> **Disclaimer**:
> I first discovered the MOVE design at https://cirw.in/blog/time-to-move-on
> I did not invent it.  I did not design it.  I simply like it.

## What is MVC

[**MVC**](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) stands for:

* Model
* View
* Controller

There is a lot of good information on MVC out there.  So I'm not going to spend much time describing it.

## What is MOVE

**MOVE** stands for:

* Model
* Operations
* Views
* Events

The major difference from **MVC** is that the controller is split in two.

* Operations: Things that this object can do
* Events: Things done to this object by another object

## MOVE by example

When describing object oriented code, I prefer to use living things as examples.  The computer science standard "Circle" to "Sphere" examples are a bit dry and too simplistic.  In my mind they are more confusing than helpful.  Why should I make an object out of circle or sphere when 3 function calls will suffice?

With that in mind, I'm going to demo the MOVE paradigm with a cat object.  Cats have their own ideas about what they will and will not do.  Knowing what a cat can and cannot do doesn't give you any insight into why - or often how - this will be accomplished.

My very loose demo code will be in `python`.

## The Cat object

### Model

The model should contain all the attributes and properties you can use.  The Operations, Views, and Events should use what is defined here.  You can add sanity checks to ensure properties are being set in a valid way.

Basically, if you expect this property/attribute in any cat, it should be in the model.

#### Example Code
``` python
class CatModel(object):
  def __init__(self):
      ## __init__ is the python object constructor
      # Can't set a default name for a cat
      self.name = None
      # Can't pick a default sex for a cat, but it should have one
      self.sex = None
      # By default cats come with repoductive capabilities
      self.fixed = False
      # By default, cats are not pregnant
      self.pregnant = False
      # Stripes? Solids? Calico?
      self.markings = None
      # Is the cat happy? sleepy? grumpy? playful?  Default to sleepy because cat.
      self.mood = 'sleepy'
      # Where is this cat?  Who knows.....
      self.location = None
      # Is the cat hidden?
      self.hidden = True
  def is_pregnant(self):
    # This is an example of validation
    if self.sex == 'M':
      raise NotImplementedError('Male cats cannot be pregnant')
    if self.fixed:
      # Already checked for 'is male', so a fixed female cat isn't pregnant
      return False
    return self.pregnant
```

A better example would use `@property` and set a bunch of validators.  But that would probably confuse folks who can't read python.  This is example code - not production code.

### Operations

Operations covers things this object (cat) can do.

``` python
class CatOperations(object):

  def jump(self, onto):
    # change location to where we have jumped
    self.location = 'on ' + str(onto)
    self.hidden = False
  def meow(self):
    return 'meow'
  def purr(self):
    return 'purr'
  def hide(self, under):
    # change location and take and consider sleeping
    self.location  = 'under' + str(under)
    self.hidden = True
    if self.mood != 'scared':
      self.mood = 'sleepy'
      self.purr()
  def attack(self, nemesis):
    if self.mood not in ['grumpy', 'playful']:
      # If the cat isn't in an attack mood, switch to one
      self.mood = 'playful'
    # jump onto nemesis
    self.jump(nemesis)
    # Call the 'kill' method/event on our nemesis and save the result
    if self.mood == 'grumpy':
      is_dead = nemesis.kill()
      if is_dead:
        # Sing the song of victory
        return self.meow()
    return ''
```

My example cat can't do very much.  But as you can see this only covers things the cat actually does on its own.

Some operations call other operations.  Sometimes the return value changes depending on the state of the cat.

The results depend on the cat.

If I wanted to make the cat able to do something else, or see why it did something, I would look here.

### Views

If I wanted to show the cat to the user running the program, here is where I would do that.

``` python
class CatViews(object):
  __str__(self):
    # in python __str__ controls how this would look as a string
    if self.hidden:
      return 'You cannot see me (I am ' + self.location ')'
    else:
      return 'I am ' + self.location + ' and am ' + self.mood
```

This is really the only acceptable place to put print statements.  Right now this is pretty boring, but there isn't much change between the **MVC** Views and **MOVE** Views.

### Events

Here is where things are done to the cat (with or without permission).

``` python
class CatEvents(object):
  def fix(self):
    # The vet removes my "bits"
    self.fixed = True
  def pet(self):
    # Humans show me love
    if self.hidden:
      # How did you find me!  Booo
      self.meow()
      self.mood = 'grumpy'

    if self.mood == 'grumpy':
      # I'm grumpy, don't pet me
      self.jump('elsewhere')
    else:
      self.mood = 'happy'
      self.purr()
```

Events "just happen" to the cat.  The cat can respond or not.  But the key quality here is that the cat recieves the action here.  With the **Operations**, the cat does the action.

Production ready code would probably be signal driven.

For folks doing web app development, I'd consider the Events to be you public API.  End users do a thing to you application.  Your application must decide what to do about that.  It would probably then return a view with the changes.

## Why is this better than MVC?

### "How do I use this class?"

The first question everyone askes when given a new object/library/etc is "how do I use it?"

With **MVC** you can look at the Controller or Views for a guide, but you are left to determine the workflow on your own.  You generally get all the things the class can do or recieve without a clear seperation between them.

With **MOVE** you should start reviewing an object based on what you want to accomplish.  Does the object do something for you (Operation) or recieve something you've done (Event).

### "How do I support this class?"

I feel like using the **MOVE** layout you are encouraged to ask better questions.  In order to find out where the problem is you need specific answers that are driven by the workflow.

Users know their workflow; developers know their code.  **MOVE** creates a common language to bridge that gap where they can speak to each other.

### New Features
You've just seen a really basic cat object.  But you can already determine where new code would go.

A senior developer asks you to add the following features to the class:

* Add support for Catnip
* Add support for self-cleaning
* Add ASCII Art cat
* Add support for cat eye color

Do you know where you would put these?

* Catnip is given to cats as a fun event
* Cat cleaning itself is an operation
* ASCII Art is something users view
* Every cat can have eye color in the model

### Bug Hunting

Lets say you've got a production application that manages healthcare records for a large country.  You've recieved the following bugs to fix:

> Dr. FakeName cannot change prescription information on any paitent.

This sounds like a permissions issue.  Where would you find the permission you need to add for Dr. FakeName?  It is probably listed under the `Events` for a paitent.

> Jane Smith shows an out of date address.

This seems like a Views related problem.  But what if she actually is unable to change her address due to a permissions bug?  Well then that is a Operations issue.
