---
title: Design Pattern Day-01
category: Programming
excerpt: |
  Java Programming Design Pattern Practice.
feature_text: |
  ## Design Pattern Day-01
  [번역] Java Programming Design Pattern Practice.
feature_image: "https://cdn-images-1.medium.com/max/1600/1*4lAASSCG5h2QLClrjj-ClA.jpeg"
image: "https://cdn-images-1.medium.com/max/1600/1*4lAASSCG5h2QLClrjj-ClA.jpeg"
---

---
## Design Pattern Day-01
---

### Abstract Document Pattern

  * <b>Intent: Achieve flexibility of untyped languages and keep the type-safety</b>

  * <img src="https://upload.wikimedia.org/wikipedia/commons/0/0e/Abstract-document-pattern.svg">

  * Usage: store variables like configuration settings in an untyped tree structure and operate on the documents using typed views.
    - The advantage of this is a <b>more loosely coupled system</b>, but it also <b>increases the risk of casting errors as the inherit type of a property is not always certain</b>

    - Example
    ```java
    public interface Document {
        Void put(String key, Object value);
        Object get(String key);
        <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor);
    }
    ```
    ```java
    public abstract class AbstractDocument implements Document {

        private final Map<String, Object> properties;

        protected AbstractDocument(Map<String, Object> properties) {
          Objects.requireNonNull(properties, "properties map is required");
          this.properties = properties;
        }

        @Override
        public Void put(String key, Object value) {
          properties.put(key, value);
          return null;
        }

        @Override
        public Object get(String key) {
          return properties.get(key);
        }

        @Override
        public <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor) {
          Optional<List<Map<String, Object>>> any = Stream.of(get(key)).filter(el -> el != null)
              .map(el -> (List<Map<String, Object>>) el).findAny();
          return any.isPresent() ? any.get().stream().map(constructor) : Stream.empty();
        }

        @Override
        public String toString() {
          StringBuilder builder = new StringBuilder();
          builder.append(getClass().getName()).append("[");
          properties.entrySet()
              .forEach(e -> builder.append("[").append(e.getKey()).append(" : ").append(e.getValue()).append("]"));
          builder.append("]");
          return builder.toString();
        }
    }
    ```

### Abstract Factory Pattern

  * <b>Intent: Provide an interface for creating families of related or dependent objects without specifying their concrete classes.</b>

  * <img src="https://upload.wikimedia.org/wikipedia/commons/a/aa/W3sDesign_Abstract_Factory_Design_Pattern_UML.jpg">

  * Usacase
    - selecting to call the appropriate implementation of FileSystemAcmeService or DatabaseAcmeService or NetworkAcmeService at runtime.
    - unit test case writing becomes much easier


### Adapter Pattern (Wrapper Pattern)

  *  Adapting between classes and objects.
  * <b>Intent: Convert the interface of a class into another interface clients expect</b>

  * <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d7/ObjectAdapter.png/300px-ObjectAdapter.png">

  * Usage: wrapper must respect a particular interface and must support polymorphic behavior.
    - use an existing class, and its interface does not match the one you need
    - create a reusable class that cooperates with unrelated or unforeseen classes, that is, classes that don't necessarily have compatible interfaces

    - Example
        ```java
          /**
           * The interface expected by the client.<br>
           * A rowing boat is rowed to move.
           *
           */
          public interface RowingBoat {
            void row();
          }
        ```
        ```java
        /**
         *
         * Adapter class. Adapts the interface of the device ({@link FishingBoat}) into {@link RowingBoat}
         * interface expected by the client ({@link Captain}).
         *
         */
        public class FishingBoatAdapter implements RowingBoat {
            private FishingBoat boat;

            public FishingBoatAdapter() {
              boat = new FishingBoat();
            }

            @Override
            public void row() {
              boat.sail();
            }
        }
        ```

### Aggregator Microservices Pattern

  * <b>Intent: The user makes a single call to the Aggregator, and the aggregator then calls each relevant microservice and collects the data, apply business logic to it, and further publish is as a REST Endpoint.</b>

  <img src="http://java-design-patterns.com/patterns/aggregator-microservices/etc/aggregator-microservice.png">

  * Usacase
    - need a unified API for various microservices, regardless the client device
