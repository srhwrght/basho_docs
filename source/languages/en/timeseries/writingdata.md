---
title: Writing Data to Riak TS
project: timeseries
version: 1.0.0+
document: guide
toc: true
index: true
audience: beginner
---

Now that you've [configured][configuring]and [activated][activating] a bucket, you are ready to write data to it.

## TODO: TOPIC HEADER

Let's use the example bucket from earlier:

```sql
CREATE TABLE GeoCheckin
(
  myfamily    varchar   not null,
  myseries    varchar   not null,
  time        timestamp not null,
  weather     varchar   not null,
  temperature float,
PRIMARY KEY (
    (quantum(time, 15, 'm'), myfamily, myseries),
    time, myfamily, myseries

           )
)
```

Riak TS allows you to write multiple rows of data at a time. Simply put the data in a list:

```ruby
client = Riak::Client.new 'myriakdb.host', pb_port: 10017
submission = Riak::TimeSeries::Submission.new client, "GeoCheckin"
submission.measurements = [["family1", "series1", 1234567, "hot", 23.5], ["family2", "series99", 1234567, "windy", 19.8]]
submission.write!
```

```erlang
{ok, Pid} = riakc_pb_socket:start_link("myriakdb.host", 10017).
riakc_ts:put(Pid, "GeoCheckin", [[“family1”, “series1”, 1234567, “hot”, 23.5], [“family2”, “series99”, 1234567, “windy”, 19.8]]).
```

```java
RiakClient client = RiakClient.newClient(10017, "myriakdb.host");
List<Row> rows = Arrays.asList(
    new Row(new Cell("family1"), new Cell("series1"), 
            Cell.newTimestamp(1234567), new Cell("hot"), new Cell(23.5)),
    
    new Row(new Cell("family2"), new Cell("series99"),
            Cell.newTimestamp(1234567), new Cell("windy"), new Cell(19.8)));

Store storeCmd = new Store.Builder("GeoCheckin").withRows(rows).build();
client.execute(storeCmd);
```

In the absence of information about which columns are provided, writing data assumes that columns are in the same order they've been declared in the table.

The timestamps should be in UTC microseconds.

If all of the data are correctly written the response is:
`ok` in Erlang, and will not raise an error in Ruby.

If some of the data failed to write, **??** do we really get an `RpbErrorResp`
with a number that failed but no guidance about which ones??!

**??** What happens if it doesn't work? What are some things the user could look at?

## Error conditions

There are two error conditions:

* Writing data to a TS bucket that doesn’t exist
*	Writing data which doesn’t match the specification of the TS bucket