---
published: draft
layout: post
category: EventStore, Nancy
title: Event Store - with examples in NancyFx
---

There's a new kid on the document database block it it's name is [Event Store][0]. Event store is an open source document database with a difference; it's documents are immutable. Documents (or events) are saved to a stream

<!--excerpt-->

## Setting up an Event Store server

There is no in memory offering from Event Store so if you want to develop against it you're going to need to set up a local server. Head over to the [Event Store downloads][0] page and grab the latest stable release. Within the package there is the EventStore.SingleNode executible. This is what you want to run. Note that this needs admin privilages in order to setup the HTTP server. This will setup your Event Store server at [http://127.0.0.1:2113/][2].

## Hooking it up to NancyFX

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

## Conclusion

Event Store gives us a fresh perspective on how to handle our data. It may be a niche perspective that wont always be appropriate for your application but it is another option that you can consider when at the drawing board. Event Store has a lot going for it but it is still young, there are a couple of things that - in my humble opinion - could really improve adoption.

1.  Improved documentation - The documentation is community submitted and hosted on Github so this one is down to all of us. The documentation on here is good but it could do with there being a lot more.
2.  Plug and play - Getting the local server set up is a bit of a faff at the moment. Event source could benefit from a Raven style in memory version for development that can be pulled in from NuGet.

   [0]: (http://geteventstore.com "Get Event Store")
   [1]: (http://download.geteventstore.com/ "Event Store downloads")
   [2]: (http://127.0.0.1:2113/)