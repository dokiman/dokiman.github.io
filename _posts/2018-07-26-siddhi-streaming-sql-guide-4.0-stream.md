---
title: Siddhi Streaming SQL Guide 4.0 (Stream ~ Query: Limit)
category: Open Source
excerpt: |
  Siddhi Streaming SQL Guide 4.0 (Stream ~ Query: Limit)
feature_text: |
  ## Siddhi Streaming SQL Guide 4.0 (Stream ~ Query: Limit)
feature_image:
image:
---

### Siddhi Streaming SQL Guide 4.0

##### Mainly Siddhi Elements (1)

- Stream
      logical series of events ordered in time.

      * syntax
      define stream <stream name> (<attribute name> <attribute type>, <attribute name> <attribute type>, ...)

      * example
      define stream TempStream (deviceID long, roomNo int, temp double);

    - Source
          receive events via multiple transports and in various
          data formats, and direct them into streams for processing.

          * syntax
          @source(type='source_type', static.option.key1='static_option_value1', static.option.keyN='static_option_valueN',
              @map(type='map_type', static.option_key1='static_option_value1', static.option.keyN='static_option_valueN',
                  @attributes( attributeN='attribute_mapping_N', attribute1='attribute_mapping_1')
              )
          )

          * example
          @source(type='http', receiver.url='http://0.0.0.0:8080/foo', basic.auth.enabled='true',
            @map(type='json'))
          define stream InputStream (name string, age int, country string);

    - Sink
          publish events from the streams via multiple transports to external endpoints in various data formats

          * syntax
          @sink(type='sink_type', static_option_key1='static_option_value1', dynamic_option_key1='{{dynamic_option_value1}}',
              @map(type='map_type', static_option_key1='static_option_value1', dynamic_option_key1='{{dynamic_option_value1}}',
                  @payload('payload_mapping')
              )
          )
          define stream StreamName (attribute1 Type1, attributeN TypeN);

          * example
          @sink(type='http', publisher.url='http://localhost:8005/endpoint', method='POST', headers='Accept-Date:20/02/2017',
            basic.auth.username='admin', basic.auth.password='admin', basic.auth.enabled='true',
            @map(type='json'))
          define stream OutputStream (name string, ang int, country string);

- Query

      Each Siddhi query can consume one or more streams, and 0-1 tables, process the events in a streaming manner, and then generate an output event to a stream or perform a CRUD operation to a table

      * syntax
      from <input stream>
      select <attribute name>, <attribute name>, ...
      insert into <output stream/table>

      * example

      define stream TempStream (deviceID long, roomNo int, temp double);

      from TempStream
      select roomNo, temp
      insert into RoomTempStream;

    - Query Projection

      supports the following for query projections

      * type
        1. Selecting required objects for projection

        2. Selecting all attributes for projection

            ex) select *

        3. Renaming attributes

            ex) temp as temperature

        4. Introducing the constant value

            ex) 'C' as scale

        5. Using mathematical and logical expressions

            ex) (), IS NULL, NOT, +-, etc..

    - Function: log, UUID, default, minimum, maximum, etc...
    - Filter: included in queries to filter information from input streams based on a <b>specified condition.</b>
          - syntax
          from <input stream>[<filter condition>]
          select <attribute name>, <attribute name>, ...
          insert into <output stream>

          - example
          from TempStream[(roomNo >= 100 and roomNo < 210) and temp > 40]
          select roomNo, temp

    - Window: Windows allow you to capture a subset of events based on a specific criterion from an input stream for calculation. Each input stream can only have a maximum of one window.

    - Aggregate Function: avg, sum, max, min, etc...

          - example
          from TempStream#window.time(10 min)
          select avg(temp) as avgTemp, roomNo, deviceID
          insert into AvgTempStream;

    - Group By: allows you to group the aggregate based on specified attributes.

          - example
          from TempStream#window.time(10 min)
          select avg(temp) as avgTemp, roomNo, deviceID
          group by roomNo, deviceID
          insert into AvgTempStream;

    - Having: filter events after processing the select statement.

          - example
          from TempStream#window.time(10 min)
          select avg(temp) as avgTemp, roomNo
          group by roomNo
          having avgTemp > 30
          insert into AlertStream;

    - Order By: order the aggregated result in ascending and/or descending order based on specified attributes.

          - example
          from TempStream#window.timeBatch(10 min)
          select avg(temp) as avgTemp, roomNo, deviceID
          group by roomNo, deviceID
          order by avgTemp, roomNo desc
          insert into AvgTempStream;

    - Limit: limit the number of events to be emitted from an aggregated results batch.

          - example
          from TempStream#window.timeBatch(10 min)
          select avg(temp) as avgTemp, roomNo, deviceID
          group by roomNo, deviceID
          order by avgTemp desc
          limit 2
          insert into HighestAvgTempStream;

##### Grammar
