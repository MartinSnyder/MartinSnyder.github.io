---
layout: post
title:  "Functional Wrappers for legacy APIs"
date:   2013-04-07 16:00:00 -0400
permalink: /blog/2013/08/07/functional-wrappers-for-legacy-apis/
---

There are many challenges in introducing a new technology into an existing project or organization.  To be successful, you normally need to be
able to achieve one or all of the following:
* Experiment with the new technology incrementally without incurring a massive adoption cost
* Showcase the strengths of the technology immediately
* Give a concrete benefit to the people on the front lines of the organization
<!--break-->
This article demonstrates techniques that can be used for such an introduction by examining a contrived example of building a functional API
for JDBC[^1] using Scala.  While the stated example is contrived, the inspiration for it is an enterprise integration API called the Documentum
Foundation Classes, which has many similarities to JDBC.

Our goal is to be able to write [functional programs][functional-programming] that efficiently interact with our legacy API. JDBC was chosen for this example because it
is widely accessible and lends itself to self-contained projects via embedded databases like [H2][h2]. Keep in mind that the point of this article
is not to provide a wrapping service for JDBC, but rather to illustrate how to construct similar wrappers for other legacy APIs that are far
less common, and often proprietary in nature.

Using our wrapper, we can write a short program that executes a SQL query and converts the results to JSON. Our answer, armed with the
facilities we construct here, will look like this:

{% highlight scala %}
val connectionInfo = new Jdbc.ConnectionInfo("jdbc:h2:mem:test1;DB_CLOSE_DELAY=-1")
 
val mapper = new ObjectMapper().registerModule(DefaultScalaModule)
 
def queryToJSON(conn: Jdbc.ConnectionInfo, sql: String) =
    Jdbc.withResultsIterator(conn, sql, it => mapper.writeValueAsString(it))
{% endhighlight %}

Lines 3 sets up a [Jackson][jackson] mapper which can convert Scala data structures to JSON. Lines 5-6 use the functional wrapper we created to iterate over our result set performing our desired data conversion. We can then write a console app as follows:

{% highlight scala %}
def main(args: Array[String]) {
  queryToJSON(connectionInfo, "SELECT * FROM EXAMPLE") match {
    case Success(json) => println(json)
    case Failure(e) => println(e.getMessage)
  }
}
{% endhighlight %}

So that’s where we’re headed. Now let’s take a closer look at how to get there!

We have two design goals when creating a functional wrapper for a non-functional API:
1. Hide state from your callers as much as possible
2. Reshape the API in terms of a collections API as much as possible

The first goal gets at the heart of functional languages.  A program that is “purely functional” has no state and never mutates a variable.
While we won’t be able to achieve this goal ourselves, we can still allow the callers of our wrapper to do so.

Of course, you’ll have a tough time hiding state if you can’t recognize it.  Let’s start by looking at a common code block in JDBC code written in Java:

{% highlight scala %}
Connection conn =  DriverManager.getConnection("url", "user", "pass");
try {
    ...
} finally {
    conn.close();
}
{% endhighlight %}

Bam! State is right there. The connection object might be open or closed, and the programmer has to know about this and manage it. This is
a potential source of programmer errors and more importantly, it completely screws up Java’s “memory management” model. When you have objects
like this, you can’t just wait for them to be garbage collected, so you’re basically back in C/C++ land and have to manage it yourself.

There’s a way to address this in Scala. Let’s build it up bit by bit:

{% highlight scala %}
case class ConnectionInfo(url: String, username: String = "", password: String = "")
{% endhighlight %}

This defines a structure named ConnectionInfo that represents the data necessary[^2] to instantiate a JDBC connection.

Syntactically there is not much to explain here. Case classes[^3] are special beasts in Scala, but they are not too different from immutable structures in other languages. The three member values (url, username, password) are all strings, and two of them have default values defined.

Now we can define a function:

{% highlight scala %}
def withConnection (connInfo: ConnectionInfo, f: Connection => Unit) { ... }
{% endhighlight %}

That declaration defines a new function named withConnection where:
* The first parameter is an object of type ConnectionInfo
* The second parameter is a function that takes a JDBC Connection and returns nothing (Unit is a special type which is effectively nothing)
* The function does not return a value

The second argument is the most important concept at this point. In Scala (and generally in all functional languages), a function is a
“first class object” which means that a function can be stored in a variable or passed as a parameter like any other object. Also, because
Scala is strongly typed, the compiler will make sure that only a function of that exact signature can be passed into withConnection.

A fair point can be made that you can accomplish this same thing using interfaces in Java. This is definitely true and, further, it is
definitely possible to program in a functional style in any language. The advantage of Scala is that this technique is syntactically convenient.
Using this technique in Scala reduces code size, whereas in Java it often can increase it.

Now we can examine the body of the function:

{% highlight scala %}
def withConnection (connInfo: ConnectionInfo, f: Connection => Unit) {
    val conn = DriverManager.getConnection(connInfo.url, connInfo.username,
            connInfo.password)
    try {
        f(conn)
    }
    finally {
        conn.close()
    }
}
{% endhighlight %}

This implementation is pretty much what one would expect, and looks similar to our original Java pattern. The advantage is twofold:
1. We only write this try/finally logic once for our application vs. in multiple places
2. We are hiding the existence of the connection state from our callers. They just need to operate on a connection, they will not need to know how to create or manage one.

Now let’s work on the return values of our function. In functional programming, it is frowned upon to define functions that do not return
values. This is because a function that returns no value (often called a procedure) likely manipulates state or produces some other side
effect — things that would violate purity. Of course, we want anyone to be able to invoke withConnection, so we’ll use Scala’s ability to
allow any return type

{% highlight scala %}
def withConnection [T] (connInfo: ConnectionInfo, f: Connection => T): T = {
    val conn = DriverManager.getConnection(connInfo.url, connInfo.username,
            connInfo.password)
    try {
        f(conn)
    }
    finally {
        conn.close()
    }
}
{% endhighlight %}

Not much has changed, as we would hope. Our function is now a template[^4] for generic type T. withConnection will return whatever type the
passed in function returns. This works almost identically to generics applied to methods in Java.

We have one more thing to address with this method before we move on. While Scala can handle Java-style exceptions in the expected fashion, it
is more common to use types that convey the possible return states of a function. In this case, we can apply something called the Try monad[^5]
to our try block. As a result, our function will return either Success or Failure (but will never actually throw) and the Scala compiler will
force our callers to distinctly handle the two scenarios.

Here is our final version:

{% highlight scala %}
def withConnection [T] (connInfo: ConnectionInfo, f: Connection => T): Try[T] = {
    val conn = DriverManager.getConnection(connInfo.url, connInfo.username,
            connInfo.password)
    val result = Try(f(conn))
    conn.close()
    result
}
{% endhighlight %}

We can now do the same thing for acting on a JDBC statement:

{% highlight scala %}
def withStatement [T] (connInfo: ConnectionInfo, f: Statement => T): Try[T] = {
    def privFun(conn: Connection) = {
        val stmt = conn.createStatement()
        try {
            f(stmt)
        }
        finally {
            stmt.close()
        }
    }
 
    withConnection(connInfo, privFun)
}
{% endhighlight %}

The basic mechanics are the same as our withConnection, but we have introduced two new elements worth mentioning. Firstly, within the body of
our function we have defined a new function privFun. This is no different from any other function declaration save for its scope. This
function is only visible inside of withStatement. We pass this privation function as an argument to withConnection, chaining the two methods
together.

Secondly, and more importantly, our private function invokes f, an original parameter to withStatement in the first place. This is a closure[^6]
— an important concept in functional programming. privFun isn’t just a function declaration. It is a function that also has in-scope objects
associated with it.

Completing our matched set, we provide withResultSet, which introduces no new concepts:

{% highlight scala %}
def withResultSet [T] (connInfo: ConnectionInfo, sql: String,
        f: ResultSet => T): Try[T] = {
    def privFun(stmt: Statement) = {
        val resultSet = stmt.executeQuery(sql)
        try {
            f(resultSet)
        }
        finally {
            resultSet.close()
        }
    }
 
    withStatement(connInfo, privFun)
}
{% endhighlight %}

Managing connections, statements and result sets[^7] accomplishes only part of our goal though and, and while we are saving our callers
from dealing with the management of these objects, we have still have left our callers dealing with the native API objects.

ResultSet, in particular, presents a very unwieldy interface. It combines:
* State regarding the result set
* Iteration
* Data structure of the results
* Read access to the “current” row
* Write access to the “current” row

A result set already has many characteristics of a collection, so it is an obvious candidate to be converted to the collections API. There
are three high level targets:
1. A concrete collection, like a list
2. An iterator
3. A stream

A concrete collection must be completely populated on construction. Iterators allow once-only dynamic traversal. Streams allow multiple
traversals and can be used to represent infinite collections, like the set of all positive integers.

Since we have no control over the number of rows in our ResultSets, we would run the risk of memory issues were we to choose concrete
collections. Streams are very difficult to use correctly in this scenario because of memoization[^8], which is used internally to implement
the multiple-traversal facility of streams. The once-only traversal nature shared by both Iterators and ResultSets is the key factor in
deciding which collection category is the best fit.

Here’s a simple wrapper to get us started:

{% highlight scala %}
class ResultsIterator(resultSet: ResultSet) extends Iterator[ResultSet] {
    def hasNext = resultSet.next()
    def next() = resultSet
}
{% endhighlight %}

Right away, we can spot a conflict between these two APIs. The ResultSet.next method method performs two actions. It scrolls the result set
to the next element AND it returns a boolean indicating if the end of the result set has been reached or not[^9]. It is worth noting that while
this iterator will work correctly without modification, that is only because of implementation details of Scala which we do not want to rely
on. Secondly, this issue is very hard to solve directly because if we try to move things around, we’ll run afoul of the internal state of the
result set. Without broader changes, this problem is best left alone.

Let’s tackle another problem first. Imagine a nefarious programmer decides to call “next” while iterating over our wrapper. That will
completely break our iteration mechanic. We need to shield ourselves from this by hiding the result set from the users of our iterator.
JDBC does not make this particularly easy for us. There is no mechanism to get a “row” of data from a result set so we will need to construct
such a row object ourselves.

There are two sorts of collection objects that we can consider for a row. One is an array-like structure such as a list or vector. The other
is a map. A map presents a clearer interface to our callers, because it allows the callers to reference the database columns by name rather
than their order in the query.

In order to construct our map, we need to determine the names of the columns available through the ResultSet. We can do this as follows:

{% highlight scala %}
val columnNames: Seq[String] = {
    val rsmd: ResultSetMetaData = resultSet.getMetaData
 
    for (i <- 1 to rsmd.getColumnCount) yield rsmd.getColumnName(i)
}
{% endhighlight %}

A for comprehension in Scala allows us to extract the field (the column name) from a larger structure (the results metadata). In this case
for each column index, we produce (yield) the name of the column at that index. Seq is a type that is similar to java.util.Collection or
Iterable.

Armed with the list of column names for a given result set, we can construct a name-value map with the following function:

{% highlight scala %}
private def buildRowMap(resultSet: ResultSet): Map[String, AnyRef] = {
    (
        for (c <- columnNames) yield c -> resultSet.getObject(c)
    ).toMap
}
{% endhighlight %}

Again, we use a for comprehension to perform the data translation. This time though, we use the -> operator in Scala to produce “tuples”.
This is the same as if we manually constructed a pair of (c, resultSet.getObject(c)) for each column name. Conveniently, we can convert a
collection of tuples like that into a map. The toMap routine we invoke assumes that each tuple represents a pair of (key, value).

Now we can produce an updated Iterator implementation using our new mechanism:

{% highlight scala %}
class ResultsIterator (resultSet: ResultSet) extends Iterator[Map[String, AnyRef]] {
    val columnNames = { … }
    private def buildRowMap(resultSet: ResultSet) = { ... }
 
    def hasNext = resultSet.next()
    def next() = buildRowMap(resultSet)
}
{% endhighlight %}

We now allow our callers to iterate over our results, but give them only an immutable structure to do so, so there is no risk of our callers
disturbing the state of our JDBC ResultSet object.

We are now left with the problem we passed over before, but we are now in a far better position to address the API conflict involving hasNext
and next. At the heart of the issue is that the ResultSet object has an internal state that is changing when “next” is called, so we cannot
directly attach next to the iterator method hasNext, which may be called multiple times inadvertently.

Our approach to this is to manage the state of the result set in our iterator instead. For this purpose, we define the following helper method:

{% highlight scala %}
private def getNextRow(resultSet: ResultSet) = {
    if (resultSet.next())
        Some(buildRowMap(resultSet))
    else
        None
}
{% endhighlight %}

Our helper method getNextRow gives us a better model for managing the two aspects of the information we wish to collect from the results set.
The Option[^10] structure conveys to us whether or not the next row exists. If it does exist, it can provide us with the value. Best of all, we
can interrogate it multiple times without changing the state of the option we are interrogating.

Observe the final Iterator:

{% highlight scala %}
class ResultsIterator (resultSet: ResultSet) extends Iterator[Map[String, AnyRef]]
{
    val columnNames = { … }
    private def buildRowMap(resultSet: ResultSet) = { … }
    private def getNextRow(resultSet: ResultSet) = { … }
 
    var nextRow = getNextRow(resultSet)
 
    def hasNext = nextRow.isDefined
 
    def next() = {
        val rowData = nextRow.get
        nextRow = getNextRow(resultSet)
        rowData
    }
}
{% endhighlight %}

Blindly calling “get” on the Option nextRow is a dangerous call that could throw a runtime exception. We don’t care about this issue in our
example because our next method is effectively “guarded” from misuse by hasNext. If someone blindly calls next, then they deserve whatever
they get!

Armed with our iterator, we can complete our utility API. Here is the final function that builds upon withResultSet to expose our iteration
mechanics to our callers:

{% highlight scala %}
def withResultsIterator [T] (connInfo: ConnectionInfo, sql: String,
        itFun: ResultsIterator => T): Try[T] =
 
    withResultSet(connInfo, sql, resultSet => itFun(new ResultsIterator(resultSet)))
{% endhighlight %}

That’s it! We can now write functional programs against JDBC. More importantly, we can identify non-functional characteristics in APIs that we
use every day, and develop an approach for reshaping those APIs in functional terms.

Thanks for reading. If you’re looking for more information, check out the footnotes or download the project source on [GitHub][github-project].

[^1]: Note that if this is something you actually want to, it’s been done several times already Scala [Slick][slick] is one that is definitely worth checking out if you want to apply functional programming techniques to database access
[^2]: It’s possible for advanced usage that more fields would be necessary, and those fields could be trivially added
[^3]: Further reading: [A Tour of Scala: Case Classes][case-classes]
[^4]: Further reading: [Writing generic functions that take classes as parameters][generic-functions]
[^5]: Further reading: [The Neophyte’s Guide to Scala Part 6: Error Handling With Try][try-monad]
[^6]: Further reading: [Functional Scala: Closures][closures]
[^7]: A general term for this technique is Automatic Resource Management. [Josh Suereth][josh] wrote a library named [scala-arm][arm] in this space. The project source and documentation provides all sorts of additional details on the topic.
[^8]: Further reading: [Scala Stream Memoization][memoization]
[^9]: This is exactly the sort of combination of functionality that functional programming opposes
[^10]: Further reading: [scala.Option Cheat Sheet][option]

[github-project]: https://github.com/MartinSnyder/scala-jdbc
[functional-programming]: https://en.wikipedia.org/wiki/Functional_programming
[h2]: http://www.h2database.com/html/main.html
[jackson]: https://github.com/FasterXML/jackson
[slick]: http://slick.lightbend.com/
[case-classes]: http://www.scala-lang.org/old/node/107
[generic-functions]: http://www.brentsowers.com/2011/11/writing-generic-functions-that-take.html
[try-monad]: http://danielwestheide.com/blog/2012/12/26/the-neophytes-guide-to-scala-part-6-error-handling-with-try.html
[closures]: https://gleichmann.wordpress.com/2010/11/15/functional-scala-closures/
[josh]: http://jsuereth.com/
[arm]: https://github.com/jsuereth/scala-arm
[memoization]: http://mikeslinn.blogspot.com/2012/11/scala-stream-memoization.html
[option]: http://blog.tmorris.net/posts/scalaoption-cheat-sheet/