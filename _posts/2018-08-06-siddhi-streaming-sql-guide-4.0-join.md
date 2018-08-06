---
title: Siddhi Streaming SQL Guide 4.0 (Query: Join ~ )
category: Open Source
excerpt: |
  Siddhi Streaming SQL Guide 4.0 (Query: Join ~ )
feature_text: |
  ## Siddhi Streaming SQL Guide 4.0 (Query: Join ~ )
feature_image:
image:
---

### Siddhi Streaming SQL Guide 4.0 (2)

##### Mainly Siddhi Elements (2)

- Query

    - Join: Joins allow you to get a combined result from two streams in real-time based on a specified condition

          - syntax
          from <input stream>#window.<window name>(<parameter>, ... ) {unidirectional} {as <reference>}
                   join <input stream>#window.<window name>(<parameter>,  ... ) {unidirectional} {as <reference>}
              on <join condition>
          select <attribute name>, <attribute name>, ...
          insert into <output stream>

          - type

            1. inner Join
            2. left outer join
            3. right outer join
            4. full outer join

    - Pattern: correlate events within a single stream or between multiple streams.

          - example
          from every( e1=TempStream ) -> e2=TempStream[ e1.roomNo == roomNo and (e1.temp + 5) <= temp ]
              within 10 min
          select e1.roomNo, e1.temp as initialTemp, e2.temp as finalTemp
          insert into AlertStream;

          - example (counting pattern)
          define stream TempStream (deviceID long, roomNo int, temp double);
          define stream RegulatorStream (deviceID long, roomNo int, tempSet double, isOn bool);

          from every( e1=RegulatorStream) -> e2=TempStream[e1.roomNo==roomNo]<1:> -> e3=RegulatorStream[e1.roomNo==roomNo]
          select e1.roomNo, e2[0].temp - e2[last].temp as tempDiff
          insert into TempDiffStream;

          - example (logical pattern)
          1. sends the stop control action to the regulator when the key is removed from the hotel room.

          define stream RegulatorStateChangeStream(deviceID long, roomNo int, tempSet double, action string);
          define stream RoomKeyStream(deviceID long, roomNo int, action string);

          from every( e1=RegulatorStateChangeStream[ action == 'on' ] ) ->
                e2=RoomKeyStream[ e1.roomNo == roomNo and action == 'removed' ] or e3=RegulatorStateChangeStream[ e1.roomNo == roomNo and action == 'off']
          select e1.roomNo, ifThenElse( e2 is null, 'none', 'stop' ) as action
          having action != 'none'
          insert into RegulatorActionStream;

          2. generates an alert if we have switch off the regulator before the temperature reaches 12 degrees.

          define stream RegulatorStateChangeStream(deviceID long, roomNo int, tempSet double, action string);
          define stream TempStream (deviceID long, roomNo int, temp double);

          from e1=RegulatorStateChangeStream[action == 'start'] -> not TempStream[e1.roomNo == roomNo and temp < 12] and e2=RegulatorStateChangeStream[action == 'off']
          select e1.roomNo as roomNo
          insert into AlertStream;

          3. generates an alert if the temperature does not reduce to 12 degrees within 5 minutes of switching on the regulator.

          define stream RegulatorStateChangeStream(deviceID long, roomNo int, tempSet double, action string);
          define stream TempStream (deviceID long, roomNo int, temp double);

          from e1=RegulatorStateChangeStream[action == 'start'] -> not TempStream[e1.roomNo == roomNo and temp < 12] for '5 min'
          select e1.roomNo as roomNo
          insert into AlertStream;

##### Grammar
