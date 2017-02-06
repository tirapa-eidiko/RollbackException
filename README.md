# RollbackException

Rollback Exception Strategy

You can define a rollback exception strategy to ensure that a message that throws an exception in a flow is rolled back for reprocessing. Use a rollback exception strategy when you cannot correct an error when it occurs in a flow. Usually, you use a rollback exception strategy to handle errors that occur in a flow that involve a transaction. If the transaction fails, that is, if a message throws an exception while being processed, then the rollback exception strategy rolls back the transaction in the flow. If the inbound connector is transactional, Mule delivers the message to the inbound connector of the parent flow again to reattempt processing (that is, message redelivery).

Beyond managing transactional errors, you can use a rollback exception strategy to:

Manage unhandled exceptions—?exceptions that the application fails to catch.

Put in flows in which messages require redelivery.

A rollback exception strategy has the potential to introduce an infinite loop of activity within a flow: a message throws an error, the rollback exception strategy catches the exception and rolls the message back for reprocessing; the message throws an error again, the rollback exception strategy catches the exception again, and rolls the message back for reprocessing, and so on.

To avoid this infinite loop and responsibly manage unresolvable errors, you can apply two limitations to a rollback exception strategy:

Define the maximum number of times that the rollback exception strategy attempts to redeliver the message for processing.

Define a flow to handle messages that exceed the maximum number of redelivery attempts.

Use Rollback Exception Strategy for Transactions
Unlike a catch exception strategy which always commits a failed transaction and consumes the message, a rollback exception strategy provides multiple attempts for a message to successfully move through a flow before committing a failed transaction and consuming the message.

For example, suppose you have a flow that involves a bank transaction to deposit funds in an account. You configure a rollback exception strategy to handle errors that occur in this flow; when an error occurs during processing — say, a flow-external bank account database is temporarily unavailable — the message throws an exception. The rollback exception strategy catches the exception, and rolls the message back to the beginning of the flow to reattempt processing. During the second attempt at processing, the database is back online again, and the message successfully reaches the end of the flow.

Mule attempts message redelivery when your flow uses one of the following two types of transports: transactional or reliable.

A transactional transport consumes a message as it travels through the flow, transforming it to a different object, for example, or enriching it with more data. When a message using a transactional transport throws an exception, a rollback exception strategy rolls back the transaction so the transport can return a message to its original state for reprocessing.

A reliable transport does not consume a message as it travels through the flow until it can ascertain that the message has successfully reached the end of the flow.When a message using a reliable transport throws an exception, the rollback exception strategy discards the partially processed message and instructs the flow to attempt processing the original message again.

If your flow involves a reliable transport and a rollback exception strategy, you must set the flow’s Processing Strategy to "synchronous". (Configure the Processing Strategy in the Flow Properties panel.) This is because reliable transports do not consume messages, they wait for messages to successfully complete a flow before consuming the messages; therefore, the flow must process synchronously.

If you deploy a Mule application that contains a flow with an asynchronous processing strategy and a rollback exception strategy configured for message redelivery, the application fails!
Transport	Type	When an exception occurs…?
VM
transactional
The message rolls back to its original state for reprocessing.
JDBC
transactional
The message rolls back to its original state for reprocessing.
JMS
transactional
The message rolls back to its original state for reprocessing.
JMS
reliable
The session recovers, that is, Mule discards the copy of the source file and attempts reprocessing the message using a fresh copy.
FTP
reliable
Mule discards the copy of the source file and attempts reprocessing the message using a fresh copy.
File
reliable
Mule discards the copy of the source file and attempts reprocessing the message using a fresh copy.
IMAP
reliable
Mule does not move the message from the mailbox and attempts reprocessing the message using a fresh copy of the original.
Use Rollback Exception Strategy for Unhandled Exceptions
At times, message processors in a flow encounter exceptions that they are unable to resolve by performing corrective measures; these types of errors are called "unhandled exceptions." Typically, all unhandled exceptions are handled by Mule’s default exception strategy, but you can customize a rollback exception strategy to more efficiently route messages with unhandled exceptions.

Depending on the flow exchange pattern, a rollback exception strategy routes messages with unhandled exceptions in one of two ways:

When the flow exchange pattern is one-way, the rollback exception strategy instructs the inbound connector transport to execute corrective actions.

When the flow exchange pattern is request-response, rollback exception strategy changes the payload of a message and returns it to the client.

Rollback Exception Strategy vs. Default Exception Strategy
Where once you would have used Mule’s default exception strategy to process unhanded exceptions, you can now configure the much improved rollback exception strategy to do so. To demonstrate the advantages rollback exceptions strategy has to offer, it is worthwhile noting some key differences in the methods each strategy employs when routing messages with unhandled exceptions.

Default Exception Strategy:

Stores the exception information in the payload of the message.

Returns null in the exceptionPayload attributeMessage.

Injects NullPayload as the message’s payload; unable to customize.

Records exception information in the exceptionPayload attribute; unable to customize.

Rollback Exception Strategy:

Retains the information in the message payload at the time the exception was thrown; does not alter the message payload.

Stores the exception information in the exceptionPayload.

Returns the message processing result during execution of the exception strategy.

Records exception information in the exceptionPayload attribute; able to customize.

Where the default exception strategy faltered, the rollback exception strategy performs. Using a rollback exception strategy, you can send messages with unhandled exceptions to a dead letter queue, send failure notifications, and change the result of a flow’s execution.