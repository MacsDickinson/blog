---
published: draft
layout: post
category: EventStore, Nancy
title: When Nancy met Event Store
---

There's a new kid on the document database block it it's name is [Event Store][0]. Event store is an open source document database with a difference; it supports the concept of event sourcing.

In this post I am going to take a look at hooking Event Store into a simple Nancy web project using the Event Store .Net API.

1.  [What is Event Sourcing?](#event-sourcing)
2.  [Setting up a local Event Store server](#server-setup)
3.  [Hooking it up to NancyFX](#nancy) 
4.  [Managing Streams](#streams)
5.  [Conclusion](#conclusion)

![Event Store Logo][3]

<!--excerpt-->

<h2 id="event-sourcing">What is Event Sourcing?</h2>

Event sourcing is an approach to data persistance different to that of a relational or a traditional document database. Instead of persisting the state of your data you maintain a sequence of events that have taken place and play them back to get the current state. Recorded events are immutable therefor if the state is updated another event is recorded and if and object is deleted you simply record an event that describes this. There are a number of different approaches that can be taken from this concept, each with their own benefits. For more detailed information on event sourcing see [this post on the event store wiki][4].

<h2 id="server-setup">Setting up a local Event Store server</h2>

There is no in memory offering from Event Store so if you want to develop against it you're going to need to set up a local server. Head over to the [Event Store downloads][0] page and grab the latest stable release. Within the package there is the EventStore.SingleNode executible. This is what you want to run. Note that this needs admin privilages in order to setup the HTTP server. This will setup your Event Store server at [http://127.0.0.1:2113/][2].

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

With this in place we can inject our connection into our modules and read and write from the desired streams.

<h2 id="streams">Managing Streams</h2>

	public class BlameModule : NancyModule
	{
		private readonly IEventStoreConnection _eventStoreConnection;

		public BaseModule(IEventStoreConnection connection)
		{
			_eventStoreConnection = connection;

			Get["/GetBlames"] = ಠ_ಠ =>
				{
					var stream = _eventStoreConnection.ReadStreamEventsBackward("Blames", -1, int.MaxValue, true);
					return stream.Events.Select(streamEvent => streamEvent.Event.Data.ParseJson<Blame>());
				};

			Post["/Add"] = ಠ_ಠ =>
				{
					var model = this.Bind<Blame>();

					var eventData = new List<EventData>
						{
							new EventData(Guid.NewGuid(), "Blame", true, model.ToJsonBytes(), null)
						};
					_eventStoreConnection.AppendToStream("Blames", ExpectedVersion.Any, eventData);
					return null;
				};
		}
	}

<h2 id="conclusion">Conclusion</h2>

Event Store gives us a fresh perspective on how to handle our data. It may be a niche perspective that wont always be appropriate for your application but it is another option that you can consider when at the drawing board. Event Store has a lot going for it but it is still young, there are a couple of things that - in my humble opinion - could really improve adoption.

1.  Improved documentation - The documentation is community submitted and hosted on Github so this one is down to all of us. The documentation on here is good but it could do with there being a lot more.
2.  Plug and play - Getting the local server set up is a bit of a faff at the moment. Event source could benefit from a Raven style in memory version for development that can be pulled in from NuGet.

   [0]: http://geteventstore.com "Get Event Store"
   [1]: http://download.geteventstore.com/ "Event Store downloads"
   [2]: http://127.0.0.1:2113/
   [3]: /../images/event-store.png "event store logo"
   [4]: https://github.com/eventstore/eventstore/wiki/Event-Sourcing-Basics "Event Sourcing Basics"