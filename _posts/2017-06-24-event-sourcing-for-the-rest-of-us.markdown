---
layout: post
title:  "Event Sourcing for the rest of us"
date:   2017-06-24 00:00:00 +0930
---

ourcing, The CommandBus or Repository of 2017? Surly you've heard about it, but what exactly is it?

Event sourcing can be described as storing the events that happened in your system in the order they happened, in some kind of store. These events are then replayed to recreate state in your system, rather than just having a single row in a table, using event sourcing, you have a full history of actions that happened in your system, and how your state got to the given point that it is in.

Let's take an example of a single entity, and what's involved with working with that using Event Sourcing. Please note that this code is strictly shown as an example on what is involved with working with Event Sourced systems on a basic level.

First off we need our EventStore, this needs to support retrieving and saving events for a given Aggregate, an Aggregate is just an Entity that events are triggered through.

{% highlight php %}
interface EventStore {
    public function save($aggregateId, $event);
    public function eventsForAggregate($aggregateId);
}
{% endhighlight %}

It's up to you where these events are stored, it could be in a row in a table in a relational database, a document store like MongoDB.

There is a database designed for storing and processing event, [https://geteventstore.com](GetEventStore), which is quite powerful and makes a lot of decisions for you.

Up next we need an event, let's say our system needs to register a Member.

{% highlight php %}
class MemberWasRegistered {

    public $memberId;
    public $firstname;
    public $surname;
    public $registeredAt;

    public function __construct($memberId, $firstname, $surname, $registeredAt) {
        $this->memberId = $memberId;
        $this->firstname = $firstname;
        $this->surname = $surname;
        $this->registeredAt = $registeredAt;
    }

}
{% endhighlight %}

So, using event sourcing, our state is built up from events, so naturally, our entities will only deal with events.

{% highlight php %}
class Member {

    private $memberId;

    private $firstname;

    private $surname,

    private $registeredAt;

    public static function register($memberId, $firstname, $surname, $registeredAt)
    {
        $member = new static;
        $member->raise(new MemberWasRegistered($memberId, $firstname, $surname, $registeredAt));
        return $member;
    }

    public function apply($event)
    {
        switch($event::class) 
        {
            case 'MemberWasRegistered':
                $this->memberId = $memberWasRegistered->getMemberId();
                $this->firstname = $memberWasRegistered->getFirstname();
                $this->surname = $memberWasRegistered->getSurname();
                $this->registeredAt = $memberWasRegistered->getRegisteredAt();
                break;
        }
    }

    public function raise($event)
    {
        $this->events[] = $event;
    }

    public function releaseEvents()
    {
        $events = $this->events;

        $this->events = [];

        return $events;
    }

    public static function rebuildFromEvents($events)
    {
        $member = new static;

        foreach($events as $event)
        {
            $this->apply($event);
        }

        return $member;
    }

}
{% endhighlight %}

You'll notice that there are five methods inside this class:

- **register** - Gets called by your application code to register a member, this fires an event internally to the entity.
- **raise** - Raise or fire an event.
- **apply**- Takes an event, and applies it to the entity, building its internal state.
- **releaseEvents** - Returns you an array of all the events that were raised internally in your entity, and clears the pendingEvents.
- **rebuildFromEvents** - Given an array of events, apply them to the entity to recreate the current state, or state up into a given point in time.


You would most likely extract this functionality to a trait, or a base class like EventSourcedEntity.

Last but not least, let's have a repository so we have a way to find and save our Member entities, this will work with the help of our event store.

Do note that every time you call find, you will be rebuilding your entity from all your events, which could get quite slow when you have lots of events, in the next post we'll go through a solution for this, called Projections.

{% highlight php %}
class MemberRepository {

    private $eventStore;

    public function __construct(EventStore $eventStore)
    {
        $this->eventStore = $eventStore;
    }

    public function find($memberId)
    {
        $events = $this->eventStore->eventsForAggregate($memberId);

        return Member::rebuildFromEvents($events);
    }

    public function save(Member $member)
    {
        $events = $member->releaseEvents();

        foreach($events as $event)
        {
            $this->eventStore->save($member->memberId, $event);
        }
    }
}

{% endhighlight %}

Let's hope that seeing some code has helped you grasp some of the moving parts of event sourcing, stay tuned for more posts on the topic, and a deeper explanation into how a system scales using event sourcing.
