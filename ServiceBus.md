# ServiceBus
## .NET
### Receving by using CreateReceiver

```c#
ServiceBusClient _client = new ServiceBusClient(connection_string);

        ServiceBusReceiver _receiver = _client.CreateReceiver(queue_name,new ServiceBusReceiverOptions() {ReceiveMode= ServiceBusReceiveMode.ReceiveAndDelete });
        var _messages =  _receiver.ReceiveMessagesAsync(2);
        foreach (var _message in _messages.Result)
        {
            Console.WriteLine($"The Sequence number is {_message.SequenceNumber}");
            Console.WriteLine(_message.Body);
            
        }

```

### Receving by using QueueClient
```c#
static async Task QueueFunction()
{
	_client = new QueueClient(_bus_connectionstring, _queue_name);

	var _options = new MessageHandlerOptions(ExceptionReceived)
	{
	    MaxConcurrentCalls = 1,
	    AutoComplete = false
	};

	_client.RegisterMessageHandler(Process_Message, _options);
    Console.ReadKey();
    }


static async Task Process_Message(Message _message,CancellationToken _token)
{
    Console.WriteLine(Encoding.UTF8.GetString(_message.Body));


    await _client.CompleteAsync(_message.SystemProperties.LockToken);
}

static Task ExceptionReceived(ExceptionReceivedEventArgs args)
{
    Console.WriteLine(args.Exception);
    return Task.CompletedTask;
}

}
```

### Send by using QueueClient

```c#
static async Task Main(string[] args)
{
    IQueueClient _client;
    _client = new QueueClient(_bus_connectionstring, _queue_name);
    Console.WriteLine("Sending Messages");
    for (int i = 1; i <=10; i++)
    {
	Order obj = new Order();
	var _message = new Message(Encoding.UTF8.GetBytes(obj.ToString()));
	await _client.SendAsync(_message);
	Console.WriteLine($"Sending Message : {obj.Id} ");
    }
}
```

## Filters
Service Bus supports three filter conditions:

_SQL Filters_ - A SqlFilter holds a SQL-like conditional expression that is evaluated in the broker against the arriving messages' user-defined properties and system properties. All system properties must be prefixed with sys. in the conditional expression. The SQL-language subset for filter conditions tests for the existence of properties (EXISTS), null-values (IS NULL), logical NOT/AND/OR, relational operators, simple numeric arithmetic, and simple text pattern matching with LIKE.

_Boolean filters_ - The TrueFilter and FalseFilter either cause all arriving messages (true) or none of the arriving messages (false) to be selected for the subscription. These two filters derive from the SQL filter.

_Correlation Filters_ - A CorrelationFilter holds a set of conditions that are matched against one or more of an arriving message's user and system properties. A common use is to match against the CorrelationId property, but the application can also choose to match against the following properties:

ContentType
Label
MessageId
ReplyTo
ReplyToSessionId
SessionId
To
any user-defined properties.
A match exists when an arriving message's value for a property is equal to the value specified in the correlation filter. For string expressions, the comparison is case-sensitive. When specifying multiple match properties, the filter combines them as a logical AND condition, meaning for the filter to match, all conditions must match.

All filters evaluate message properties. Filters can't evaluate the message body.
