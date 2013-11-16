---
published: draft
layout: post
category: EventStore, Nancy
title: When Nancy met Event Store
---

There's a new kid on the document database block it it's name is [Event Store][0]. Event store is an open source document database with a difference; it supports the concept of event sourcing.

In this post I am going to take a look at hooking Event Store into a simple Nancy web project using the Event Store .Net API.

![Event Store Logo][3]

<!--excerpt-->

1.  [What is Event Sourcing?](#event-sourcing)
2.  [Setting up a local Event Store server](#server-setup)
3.  [Hooking it up to NancyFX](#nancy) 
4.  [Managing Streams](#streams)
5.  [Conclusion](#conclusion)

<h2 id="event-sourcing">What is Event Sourcing?</h2>

Event sourcing is an approach to data persistance different to that of a relational or a traditional document database. Instead of persisting the state of your data you maintain a sequence of events that have taken place and play them back to get the current state. Recorded events are immutable therefor if the state is updated another event is recorded and if and object is deleted you simply record an event that describes this. There are a number of different approaches that can be taken from this concept, each with their own benefits. For more detailed information on event sourcing see [this post on the event store wiki][4].

<h2 id="server-setup">Setting up a local Event Store server</h2>

There is no in memory offering from Event Store so if you want to develop against it you're going to need to set up a local server. Head over to the [Event Store downloads page][0] and grab the latest stable release. Within the package there is the EventStore.SingleNode executible. This is what you want to run. Note that this requires admin privilages in order to setup the HTTP server. Once this is running you can visit the http management studio at [http://127.0.0.1:2113/][2]. There is a lot of goodness in here; you can run through an example implementation, take a look at stored streams, set up projections, query the data, view live stats and so on. However; in order to do anything useful you will need to log in as the admin. The default password is changeme. You should change this once logged on.

<h2 id="nancy">Hooking it up to NancyFX</h2>

Plugging the .NET client API into a VS project is easy, it's just a case of pulling down the EventSource.Client package:

	PM> Install-Package EventStore.Client

With this installed you can look at creating a connection provider for EventStore. This will be responsible creating the connection to the TCP end point.

    public class EventStoreConnectionProvider
    {
        private const int _tcpIpPort = 1113;

        private static IEventStoreConnection _eventStoreConnection;

        public static IEventStoreConnection EventStore
        {
            get { return _eventStoreConnection ?? (_eventStoreConnection = CreateEventStoreConnection()); }
        }

        private static IEventStoreConnection CreateEventStoreConnection()
        {
            var  tcpEndPoint = new IPEndPoint(IPAddress.Parse("127.0.0.1"), _tcpIpPort);
            
            return EventStoreConnection.Create(ConnectionSettings.Default, tcpEndPoint);
        }
    }

Once you've got this you're going to want to open a connection on application start and drop it into your IoC container.

	protected override void ConfigureApplicationContainer(Nancy.TinyIoc.TinyIoCContainer container)
	{
		base.ConfigureApplicationContainer(container);
		
		var connection = EventStoreConnectionProvider.EventStore;
		connection.Connect();
		container.Register(connection);
	}

<h2 id="streams">Managing Streams</h2>

With the connection in our IoC container we can simply inject it into our modules where we can interact with our desired streams.

	public class BlameModule : NancyModule
	{
		private readonly IEventStoreConnection _connection;

		public BaseModule(IEventStoreConnection connection)
		{
			_connection = connection;

		}
	}

When reading we take events from the top of the stack, most recent first. In this example I am requesting all events (-1 to int.MaxValue) from the "Blames" stream and then parsing them to the Blame object.

    Get["/GetBlames"] = __ =>
        {
            var stream = _connection.ReadStreamEventsBackward("Blames", -1, int.MaxValue, true);
            return stream.Events.Select(x => x.Event.Data.ParseJson<Blame>());
        };

Writing to a stream is also straight forward. We save our object as EventData, this holds the id, type, data and meta data for your model. We then append the EventData to the corresponding stream.

    Post["/Add"] = __ =>
        {
            var model = this.Bind<Blame>();

            var eventData = new List<EventData>
                {
                    new EventData(Guid.NewGuid(), "Blame", true, model.ToJsonBytes(), new { DateAdded = DateTime.Now }.ToJsonBytes())
                };
            _eventStoreConnection.AppendToStream("Blames", ExpectedVersion.Any, eventData);
            return null;
        };

<h2 id="conclusion">Conclusion</h2>

Event Store gives us a fresh perspective on how to handle our data. It may be a niche angle and it wont be appropriate for every application but it is another option that can be considered when at the drawing board. Event Store has a lot going for it but it is still young, there are a couple of things that - in my humble opinion - could really boost user adoption.

1.  Improved documentation - There is a lot of information on the docs on Github but they're not the easiest read. They could really benefit with a number of step by step guides with examples, much like the Nancy docs. Documentation is community submitted on Github so adding this is down to all of us.
2.  Plug and play - Event Store could benefit from a Raven style in-memory version that could be pulled in from NuGet. Setting up a local server isn't difficult but being able to pull everything that need from NuGet really adds to the simplicity of the product.

   [0]: http://geteventstore.com "Get Event Store"
   [1]: http://download.geteventstore.com/ "Event Store downloads"
   [2]: http://127.0.0.1:2113/
   [3]: /../images/event-store.png "event store logo"
   [4]: https://github.com/eventstore/eventstore/wiki/Event-Sourcing-Basics "Event Sourcing Basics"