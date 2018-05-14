
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [What is `stream` api](#what-is-stream-api)
* [Usage of stream](#usage-of-stream)
	* [Stream pipeline](#stream-pipeline)
	* [Intermediate Operations](#intermediate-operations)
		* [distinct()](#distinct)
		* [filter(Predicate<? super T> predicate)](#filterpredicate-super-t-predicate)
		* [flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)](#flatmapfunction-super-t-extends-stream-extends-r-mapper)
		* [limit(long maxSize)](#limitlong-maxsize)
		* [map(Function<? super T, ? extends R> mapper)](#mapfunction-super-t-extends-r-mapper)
		* [peek(Consumer<? super T> action)](#peekconsumer-super-t-action)
		* [skip(long n)](#skiplong-n)
		* [sorted()](#sorted)
		* [sorted(Comparator<? super T> comparator)](#sortedcomparator-super-t-comparator)
	* [Terminal Operations](#terminal-operations)
		* [allMatch(Predicate<? super T> predicate)](#allmatchpredicate-super-t-predicate)
		* [anyMatch(Predicate<? super T> predicate)](#anymatchpredicate-super-t-predicate)
		* [collect(Collector<? super T,A,R> collector)](#collectcollector-super-tar-collector)
		* [collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)](#collectsupplierr-supplier-biconsumerr-super-t-accumulator-biconsumerr-r-combiner)
		* [count()](#count)
		* [findAny()](#findany)
		* [findFirst()](#findfirst)
		* [forEach(Consumer<? super T> action)](#foreachconsumer-super-t-action)
		* [forEachOrdered(Consumer<? super T> action)](#foreachorderedconsumer-super-t-action)
		* [max(Comparator<? super T> comparator)](#maxcomparator-super-t-comparator)
		* [min(Comparator<? super T> comparator)](#mincomparator-super-t-comparator)
		* [noneMatch(Predicate<? super T> predicate)](#nonematchpredicate-super-t-predicate)
		* [reduce(BinaryOperator<T> accumulator)](#reducebinaryoperatort-accumulator)
		* [reduce(T identity, BinaryOperator\<T> accumulator)](#reducet-identity-binaryoperatort-accumulator)
		* [reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator\<U> combiner)](#reduceu-identity-bifunctionu-super-t-u-accumulator-binaryoperatoru-combiner)
	* [static util functions](#static-util-functions)
		* [builder()](#builder)
		* [concat(Stream<? extends T> a, Stream<? extends T> b)](#concatstream-extends-t-a-stream-extends-t-b)
		* [empty()](#empty)
		* [generate(Supplier\<T> s)](#generatesuppliert-s)
		* [iterate(T seed, UnaryOperator\<T> f)](#iteratet-seed-unaryoperatort-f)
		* [of(T... values) and of(T value)](#oft-values-and-oft-value)
	* [Collectors](#collectors)
		* [averagingDouble(ToDoubleFunction<? super T> mapper)](#averagingdoubletodoublefunction-super-t-mapper)
		* [collectingAndThen(Collector<T,A,R> downstream, Function<R,RR> finisher)](#collectingandthencollectortar-downstream-functionrrr-finisher)
		* [groupingBy](#groupingby)
		* [mapping(Function<? super T, ? extends U> mapper, Collector<? super U, A, R> downstream)](#mappingfunction-super-t-extends-u-mapper-collector-super-u-a-r-downstream)
		* [toMap](#tomap)
* [More APIs for collections](#more-apis-for-collections)
	* [Map](#map)
		* [compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)](#computek-key-bifunction-super-k-super-v-extends-v-remappingfunction)
		* [getOrDefault(Object key, V defaultValue)](#getordefaultobject-key-v-defaultvalue)
		* [replaceAll(BiFunction<? super K, ? super V, ? extends V> function)](#replaceallbifunction-super-k-super-v-extends-v-function)
	* [Optional](#optional)
		* [Postpone the handle of NPE](#postpone-the-handle-of-npe)
		* [filter, flatMap and map](#filter-flatmap-and-map)
		* [an example of the usage of Optional](#an-example-of-the-usage-of-optional)

<!-- /code_chunk_output -->

# What is `stream` api

`stream` apis are added in Java8 to 
> support functional-style operations on streams of elements, such as map-reduce transformation on collections.

With `stream`, the operations on collections will be more readable. It can also provide performance improvement when using apis like `parallelStream()` to handle a large amount of data. In addition, together with some data source api, `stream`s can also save the memory usage. For example, `java.nio.file.Files#lines` can create a stream to read a fine line by line.

# Usage of stream

## Stream pipeline

Typically, `stream` apis are used with chained invocations called stream pipeline. A stream pipeline consists of a _source_ (which might be an array, a collection, a generator function, an I/O channel, etc), zero or more _intermediate operations_ (which transform a stream into another stream), and a _terminal operation_ (which produces a result or side-effect).

Following is an example of a stream usage:
```java
List<String> jobIds = scheduledScaleJobs
    .stream()
    .map(ScheduledScaleJob::getJobId)
    .collect(Collectors.toList());
```
In this stream pipeline, `scheduledScaleJobs.stream()` is the _source_, it create a `Stream<ScheduledScaleJob>` object based on the list object `scheduledScaleJobs`.

Following `stream()`, `map(ScheduledScaleJob::getJobId)` is the _intermediate operation_ which convert the `Stream<ScheduledScaleJob>` object to a `Stream<String>` object. The convertion is done as a map from `ScheduledScaleJob` object to the result of a call to the object's `getJobId` method.

At last, `collect(Collectors.toList())` is called as the _terminal operation_. This _terminal operation_ terminates the pipeline by collecting the elements in the stream with a `Collector` returned by `Collectors.toList()`. It is obviously from the name that the `Collector` will collect the elements into a `List`.

## Intermediate Operations

In this section, paramterized type `T` will be the type of object in the upstream.

### distinct()

`Stream#distinct()` will returns `Stream<T>`.

`distinct` is simple and easy to understand. The downstream contains the distinct elements from the upstream. `Object#equals(Object)` is used in it.

### filter(Predicate<? super T> predicate)

`Stream#filter(Predicate<? super T>)` will return `Stream<T>`.

`filter` is also simple to understand. Only the elements pass the predicate will be send to the downstream.

It is common to have multiple conditions which all needs to be `true`. In this case, use multiple filter is more readable then `&&`. The performance between the two way depends on the specific condition. Usually, there won't be too much difference.
Here is an example with multiple conditions:
```java
List<String> nonEmptyStrs = strs
        .stream()
        .filter(Objects::nonNull)
        .filter(str -> !"".equals(str))
        .collect(Collectors.toList());
```
The second `filter` can be replaced with (not common)
```java
        .filter(((Predicate<String>)""::equals).negate())
```
Comparing with `&&`:
```java
List<String> nonEmptyStrs = strs
        .stream()
        .filter(str -> Objects.nonNull(str) && !"".equals(str))
        .collect(Collectors.toList());
```

### flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)

`Stream#flatMap(Function<? super T, ? extends Stream<? extends R>>)` will return `Stream<R>` where R is any other type.

`flatMap` is used to 
>get a stream consisting of the results of replacing each element of the upstream with the contents of a mapped stream produced by applying the provided mapping function to each element.

Simply speaking, `flatMap` is used to map every element (whose type is `T`) in the upstream to multiple elements of type `R` (one to many mapping) and put all the mapped elements into a single stream (which "flat" means).

For example, `flatMap` can be used to get a `Character` stream from a `String` stream that all the characters in all the `String`s will be put in the downstream.
Here is an example to count how many distinct characters are there in the `String` list:
```java
long distinctCharacterCount = strs
        .stream()
        .flatMap(str -> Stream.of(str.toCharArray()))
        .distinct()
        .count();
```

Similar to `flatMap`, there are `flatMapToDouble`, `flatMapToInt` and `flatMapToLong` which are used to get `DoubleStream`, `IntStream` and `LongStream`.

### limit(long maxSize)

`Stream#limit(long)` will limit the elements in the downstream to the first `maxSize` elements in the upstream.

### map(Function<? super T, ? extends R> mapper)

`Stream#map(Function<? super T, ? extends R>)` will return `Stream<R>` where R is another type.

`map` is similar to `flatMap` except that it is one on one mapping. It is useful when you need extracting properties, converting objects, etc.

The example in the beginning shows the usage of `map`, it extract the `jobId` property from the `ScheduledScaleJob` objects:
 ```java
 List<String> jobIds = scheduledScaleJobs
        .stream()
        .map(ScheduledScaleJob::getJobId)
        .collect(Collectors.toList());
 ```

 When use map for object convertion, you can use lambda expression like this:
 ```java
List<ScheduledScaleJobPO> scheduledScaleJobPOS = 
        scaleJobsWithJobIdAndStatus
            .stream()
            .map(item -> {
                ScheduledScaleJobPO scaleJobPO = new ScheduledScaleJobPO();
                scaleJobPO.setJobId(item.getJobId());
                scaleJobPO.setJobStatus(item.getJobStatus());
                return scaleJobPO;
            }).collect(Collectors.toList());
 ```
 However, the code will be much more readable if putting the mapping logic in a well named method like this:
```java
List<ScheduledScaleJobPO> scheduledScaleJobPOS = 
        scheduledScaleJobs
            .stream()
            .map(this::scheduledScaleJobToPO)
            .collect(Collectors.toList());
```
and `scheduledScaleJobToPO` is like this:
```java
private ScheduledScaleJobPO scheduledScaleJobToPO(ScheduledScaleJob job) {
        ScheduledScaleJobPO scaleJobPO = new ScheduledScaleJobPO();
        scaleJobPO.setJobId(job.getJobId());
        scaleJobPO.setJobStatus(job.getJobStatus());
        return scaleJobPO;
}
```
Another option is use contructor of `ScheduledScaleJobPO` like this:
```java
List<ScheduledScaleJobPO> scheduledScaleJobPOS = 
        scheduledScaleJobs
            .stream()
            .map(scheduledScaleJobToPO::new)
            .collect(Collectors.toList());
```
```java
public ScheduledScaleJobPO(ScheduledScaleJob job) {
    // init the PO
}
```

Similar to `flatMap`, `flatMapToDouble`, `flatMapToInt` and `flatMapToLong`, there are also `mapToDouble`, `mapToInt` and `mapToLong`.

### peek(Consumer<? super T> action)

`Stream#peek(Consumer<? super T>)` will perform the provided action when the element consumed from the downstream.

`peek` is useful to apply some action to the elements in the middle of the pipeline.

### skip(long n)

`Stream#skip(long)` will discard the first `n` elements from the upstream.

### sorted()

`Stream#sorted()` will sort the elements in the upstream and put them in downstream according to natural order.

If the elements are not `Comparable`, `ClassCastException` will be thrown when the terminal operation is executed.

For ordered stream, the sort is stable. For unordered stream, there's no stability guaranteed.

### sorted(Comparator<? super T> comparator)

`Stream#sorted(Comparator<? super T>)` will sort the elements in the upstream and put them in downstream accroding to the comparator.

The stability is the same as `Stream#sorted()`.

## Terminal Operations

### allMatch(Predicate<? super T> predicate)

`Stream#allMatch(Predicate<? super T>)` tells whether all elements of the stream match the given predicate. If the stream is empty, the result is `true`.

This method evaluates the _universal quantification_ of the predicate over the elements of the stream ($\forall x P(x)$)

### anyMatch(Predicate<? super T> predicate)

`Stream#anyMatch(Predicate<? super T>)` tells whether any elements of the stream match the given predicate. If the stream is empty, the result is `false`.

This method evaluates the _existential quantification_ of the predicate over the elements of the stream ($\exists x P(x)$).

### collect(Collector<? super T,A,R> collector)

`Stream#collect(Collector<? super T,A,R>)` 
>performs a mutable reduction operation on the elements of this stream using a `Collector`

Simply speaking, it collects the elements into a mutable `Collection`. `R` is the type of the collection.
For the detail of the collectors, see [Collectors](#collectors) section.

### collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)

`Stream#collect(Supplier<R>, BiConsumer<R, ? super T>, BiConsumer<R, R>)` is the general collect methods. This method collects the elements in the stream into a collection with type `R` using the `supplier`, `accumulator` and `combiner`.

`supplier` is used to generate the collection. `accumulator` is used to add the element into the collections and `combiner` is used to combine two collections. The result is equivalent to
```java
R result = supplier.get();
for (T element: this stream)
    accumulator.accept(result, element);
return result;
```
As an example, following code snippets shows using the collect method to concatenate `String`s in a stream
```java
String result = strStream
        .collect(
            StringBuilder::new,
            StringBuilder::append,
            StringBuilder::append
        );
```
Another example is to collect the elements in a stream to a `ArrayList`
```java
ArrayList<String> result = strStream
        .collect(
            ArrayList::new,
            ArrayList::add,
            ArrayList::addAll
        );
```

This method is not used frequently because JDK has provided plenty of pre-defined `Collector` with `Collectors` class.

### count()

`Stream#count()` counts the number of elements in the stream.

### findAny()

`Stream#findAny()` returns an `Optional` describing some element of the stream or an empty `Optional` if the stream is empty.

### findFirst()

`Stream#findAny()` returns an `Optional` describing first element of the stream or an empty `Optional` if the stream is empty.

### forEach(Consumer<? super T> action)

`Stream#forEach(Consumer<? super T>)` performs the action for each element of the stream. Note that the consume order is not guaranteed.

### forEachOrdered(Consumer<? super T> action)

`Stream#forEachOrdered(Consumer<? super T>)` performs the action for each element of the stream in order.

### max(Comparator<? super T> comparator)

`Stream#max(Comparator<? super T>)` returns the max value in the stream according to the comparator.

### min(Comparator<? super T> comparator)

`Stream#min(Comparator<? super T>)` returns the max value in the stream according to the comparator.

### noneMatch(Predicate<? super T> predicate)

`Stream#nonMatch(Predicate<? super T>)` returns true if there's no element in the stream match the predicate. If the stream is empty, the result is `true`.

This method evaluates the _universal quantification_ of the negated predicate over the elements of the stream ($\forall x \neg P(x)$).

### reduce(BinaryOperator<T> accumulator)

`Stream#reduce(BinaryOperator<T>)` performes a reduction on the element of the stream using a *associative* accumulation function and returns an `Optional` describing the reduced value, if any.

The process is equivalent to
```java
boolean foundAny = false; 
T result = null;
for (T element: this stream) {
    if (!foundAny) {
        foundAny = true;
        result = element;
    } else {
        result = accumulator(result, element);
    }
}
return foundAny ? Optional.of(result) : Optional.empty();
```

Following is an example to use this method to add all the `int` in the stream:
```java
long result = intStream
        .reduce(Integer::sum);
```

### reduce(T identity, BinaryOperator\<T> accumulator)

`Stream#reduce(T, BinaryOperator<T>)` performs a reduction on the element of the stream using a *associative* accumulation function with a given identity.

The process is equivalent to:
```java
T result = identity;
for (T element: this stream) {
    result = accumulator.apply(result, element);
}
return result;
```

sum can also be implemented with this method by using 0 as `identity`:
```java
long result = intStream
        .reduce(0, Integer::sum);
```
String concatnate can also be achieved with `reduce`. For example, to concate all the strings in the stream to a result start with "WAP":
```java
String result = stringStream
        .reduce("WAP", String::concate);
```

### reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator\<U> combiner)

`Stream#reduce(U, BiFunction<U, ? super T>, U>, BinaryOperator<U>)` is a more complicate reduce funtion to perform a reduction with the provided identity, accumulation and combining functions.

The process is equivalent to:
```java
U result = identity;
for (T element: this stream) {
    result = accumulator.apply(result, element)
}
return result;
```
There are some restriction on the parameters:
* The `identity` must be an identity for the `combiner` as well which means `combiner(identity, u)` equals to `u` for all `u`.
* The `combiner` must be compatible with the accumulator which means `combiner.apply(u, accumulator.apply(identity, t))` equals to `accumulator.apply(u, t)` for all `u` and `t`.

Simply speaking, the execution order is not guarantee, so the `identity`, `accumulator` and `combiner` should work in all execution orders.

This method is useful for reduce the elements to a different type. For example, concanating the integers in a stream as strings:
```java
String resultStr = intStream
        .reduce(
            "",
            (result, intValue) -> result + intValue,
            String::concate
        );
```

## static util functions

### builder()

`Stream#builder()` returns a builder of a stream. Here is an example:
```java
Stream<String> strStream = Stream.builder()
        .add("HELLO")
        .add("WAP")
        .add("HUE")
        .build();
```

### concat(Stream<? extends T> a, Stream<? extends T> b)

`Stream#concat(Stream<? extends T>, Stream<? extends T>)` concate two streams into a single one. The result stream is lazily concatenated.

### empty()

`Stream#empty()` returns an empty stream.

### generate(Supplier\<T> s)

`Stream#generate(Supplier<T>)` returns an infinite sequential unordered stream where each element is generated by the provided Supplier.

### iterate(T seed, UnaryOperator\<T> f)

`Stream#iterate(T, UnaryOperator<T>)` returns an infinite sequential ordered stream produced by iterative application of a function `f` to an initial element seed, producing a stream consisting of `seed`, `f(seed)`, `f(f(seed))`, etc.

### of(T... values) and of(T value)

`Stream#of(T...)` and `Stream#of(T)` returns a stream with the given elements with the given order.

## Collectors

JDK provides a lot of `Collector`s used with `Stream#collect(Collector<? super T,A,R>)`. We can get these pre-defined `Collector`s with static method in `Collectors` class.

This section will introduce the most important and hard to understand ones. Read the [JavaDoc](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) to find more collectors.

### averagingDouble(ToDoubleFunction<? super T> mapper)

The `mapper` is used to map the element to `Double` and then calculate the average. It is equalevent to
```java
stream.map(mapper).reduce(Double::sum) / stream.count();
```

There are similar collectors as `averagingInt(ToIntFunction<? super T>)` and `averagingLong(ToLongFunction<? super T>)`.

In addition to `averaging*`, there are more collectors like `summarizing*` and `summing*` used to get summary statitics and sum.

### collectingAndThen(Collector<T,A,R> downstream, Function<R,RR> finisher)

This method is used to adapts a `Collector` to perform an additional finishing transformation. Which means the collected result will be transformed again with the finisher.
For example, to collect an immutable list:
```java
List<String> result = strStream
        .collect(collectAndThen(
            toList(),
            Collections::unmodifiableList
        ));
```

### groupingBy

`groupingBy` is used to collect the result to a map with group.
There are many types of `groupingBy` collectors:

* `groupingBy(Function<? super T, ? extends K> classifier)`: the collecting result is `Map<K, List<T>>`. Here is an example to collect strings grouped by the starting character:
```java
Map<Character, List<String>> result =
        strStream
        .collect(groupingBy(str -> str.charAt(0)));
```
* `groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T,A,D> downstream)`: the collecting result is `Map<K, D>`. This collector will use `classifier` to group the elements and use `downstream` to reduce each group. Here is an example to collect strings grouped by the starting character and concate each group into a single string:
```java
Map<Character, String> result = 
        strStream
        .collect(groupingBy(
            str -> str.charAt(0),
            joining()
        ));
```
* `groupingBy(Function<? super T,? extends K> classifier, Supplier<M> mapFactory, Collector<? super T,A,D> downstream)`: the collecting result is `M extends Map<K, D>`. In addition to the above one, `mapFactory` is used to generate the `Map` used to collect the result.

Each of them have a `groupingByConcurrent` version to used to collect as a `ConcurrentMap`

### mapping(Function<? super T, ? extends U> mapper, Collector<? super U, A, R> downstream)

`mapping` is used to adapts a collector of another type `U` to the type in the stream (`T`). The `mapper` is used to convert `T` to `U`.

### toMap

`toMap` is used to collect the elements to a map.
There are many types of `toMap` collectors:

* `toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)`: this is the most simple one. `keyMapper` is used to get key from the element and `valueMapper` is used to get value from the element. If there are duplicate keys, `IllegalStateException` is thrown.
* `toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper, BinaryOperator<U> mergeFunction>)`: this is used to handle the condition that there are duplicate keys. `mergeFunction` are used to merge the two values. For example, to collect the `String`s in a stream with a map from the length of the key to the concatation of all the strings with the same length seperated by `,`:
```java
Map<Integer, String> result =
        strStream
            .collect(toMap(
                String::length,
                Function.identity(),
                (str1, str2) -> str1 + ", " + str2
            ));
```
* `toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction, Supplier<M> mapSupplier)`: in addition to the above method, `mapSupplier` can be used to get the map used to collect elements.

`toMap` have `ConcurrentMap` versions as well.

# More APIs for collections

## Map

There are many APIs defined by `MAP<K, V>` to operate with the map datatype. It is also useful when you need handle data in a map.

### compute(K key, BiFunction<? super K,? super V,? extends V> remappingFunction)

`Map#compute(K, BiFunction<? super K, ? super V, ? extends V>)` returns the result of applying `remappingFunction` to the value. The process is equivalent to the following:
```java
V oldValue = map.get(key);
V new Value = remappingFunction.apply(key, oldValue);
if (oldValue != null) {
    if (newValue != null) {
        map.put(key, newValue);
    } else {
        map.remove(key);
    }
} else {
    if (newValue != null) {
        map.put(key, newValue);
    } else {
        return null;
    }
}
```

This method is useful to make some change to the existing key-value pair, for example, to add new employee in the list mapped by WAP:
```java
Map<Company, List<Employee>> employeeInCompanies;
Employee newWAPEmployee;
Company WAP;

employeeInCompanies.compute(
    WAP, 
    (company, employees) -> 
        employees == null ? 
            Arrays.asList(newWAPEmployee) : employees.add(newWAPEmployee)
);
```
Comparing with using old APIs:
```java
Map<Company, List<Employee>> employeeInCompanies;
Employee newWAPEmployee;
Company WAP;

List<Employee> employees = employeeInCompanies.get(WAP);
employees = 
    employees == null ? 
        Arrays.asList<newWAPEmployee) : employees.add(newWAPEmployee);
employees.put(WAP, employees);
```

There're two similar functions: 
* `Map#computeIfAbsent(K, Function? super K, ? extends V> mappingFunction)`: it is used to compute the value if not exist. The process is equivalent to:
```java
if (map.get(key) == null) {
    V newValue = mappingFunction.apply(key);
    if (newValue != null) {
        map.put(key, newValue)
    }
}
```
* `Map#computeIfPresent(K, BiFunction<? super K, ? super V, ? extends V> remappingFunction)`: it is used to compute the value if the key exists. The process is equivalent to:
```java
if (map.get(key) != null) {
    V oldValue = map.get(key);
    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue != null) {
        map.put(key, newValue);
    } else {
        map.remove(key);
    }
}
```

In addition to `compute*` methods, there's another method which is more useful. 
`Map#merge(K, V, BiFunction<? super V, ? super V, ? extends V>)` is used to insert the key-value pair to the map if key not exist (or map to `null`), or use the `BiFunction` to remap the existing value. The process is equivalent to:
```java
V oldValue = map.get(key);
V newValue = (oldValue == null) ? value : remappingFunction.apply(oldValue, value);
if (newValue == null) {
    map.remove(key);
} else {
    map.put(key, newValue);
}
```
The above emxample can be re-write with this methods like this:
```java
Map<Company, List<Employee>> employeeInCompanies;
Employee newWAPEmployee;
Company WAP;

employeeInCompanies.merge(
    WAP, 
    Arrays.asList(newWAPEmployee), 
    (l1, l2) -> {
        l1.addAll(l2);
        return l2;
    }
);
```
or if you use a util method:
```java
Map<Company, List<Employee>> employeeInCompanies;
Employee newWAPEmployee;
Company WAP;

employeeInCompanies.merge(
    WAP, 
    Arrays.asList(newWAPEmployee), 
    ListUtils::union
);
```

### getOrDefault(Object key, V defaultValue)

This method is simple, it will return `defaultValue` if the `key` no in the map.

There's another methods `Map#putIfAbsent(K key, V value)` who put the key-value into the map if the key not exist.

### replaceAll(BiFunction<? super K, ? super V, ? extends V> function)

`Map#replaceAll(BiFunction<? super K, ? super V, ? extends V>)` can be used to replace all the value in the map by applyint the `BiFunction` to each key-value pair. It is useful for works like string formatting/transformation. For example, to add "WAP" as prefix every value in the map if the key contains "WAP":
```java
Map<String, String> strMap;
strMap.replaceAll((key, value) -> {
    if (key.contains("WAP")) {
        return "WAP" + value;
    }
    return value;
});
```

## Optional

`Optional` is a container for a single object. It can also be treated as a collection except that there's only one element in the "collection"

### Postpone the handle of NPE

The most common usage of `Optional` is to postpone the handle of null pointer to a place as late as possible. You can always return `Optional<?>` to the caller if null pointer needs not to be considered in the process.
When the object in the `Optional` needs to be used, use `get`, `orElse`, `orElseGet`, `orElseThrow` to handle the null pointer condition. The usage is easy to understand according to the names:
* `get()` will return the object in the `Optional` or throw `NullPointerException` if the `Optional` is empty.
* `orElse(T)` will return the value if present or return the given value if not.
* `orElseGet(Supplier<? extends T>)` is similar to `orElse` but the default value is supplied by the `Supplier`.
* `orElseThrow(Supplier<? extends X>)` is similar to `get` but the thrown exception is supplied by the `Supplier`.

### filter, flatMap and map

`Optional` have three methods simiar to `Stream` which are as follows:
* `filter(Predicate<? super T>)` will return empty `Optional` if the value not present or the `Predicate` fails.
* `flatMap(Function<? super T, Optional<U>>)` will return apply flat map to the value and return an `Optional` of the result. The return type is `Optional<U>`.
* `map(Function<? super T, ? extends U>)` will map the value to another type use the `Function`. The return type is `Optional<U>`.

The differece between `flatMap` and `map` is that the mapper of `flatMap` returns an `Optional` directly. The result won't be wrapped to another `Opational`.

### an example of the usage of Optional

Following is the most simplest usage of `Optional`. The id of `ScaleJob` is not necessarily generated when trigger the job. So, `orElse` is used to handle the condition of id not generated jobs.
```java
// ScaleJob.java
public class ScaleJob {
    private Opationl<String> id;
    private String extraVars;

    // getters and setters
}
```
```java
// ScaleService.java
class ScaleService {
    public void scale(ScaleJob scaleJob) {
        String id = scaleJob.getId().orElse(this::newId);
        // trigger scale with the id
    }
}
```

This case is too simple so it provides no much improvement comparing this triditional null checking:
```java
// ScaleJob.java
public class ScaleJob {
    private String id;
    private String extraVars;

    // getters and setters
}
```
```java
// ScaleService.java
public class ScaleService {
    public void scale(ScaleJob scaleJob) {
        String id = scaleJob.getId();
        if (id == null) {
            id = this.newId();
        }
    }
}
```

However, consider the following example of get schdeuled time of a scale job:
```java
// ScheduledScaleService.java
public class ScheduledScaleService {
    public Long getScheduleTimeById(String id) {
        ScaleJob job = scheduledScaleMapper.getScheduledScaleJob(id);
        if (job == null) {
            throw new JobNotFoundException(id);
        } else {
            return job.getScheduledTime();
        }
    }
}
```
```java
// ScheduledScaleController.java
public class ScheduledScaleController {
    public Date getScheduledTimeById(String id) {
        try {
            return new Date(scheduledScaleService.getScheduleTimeById(id));
        } catch (JobNotFoundException e) {
            throw new JobIdNotExistException(id);
        }
    }
}
```
Another solution is do null checking:

```java
public class ScheduledScaleService {
    public Long getScheduleTimeById(String id) {
        ScaleJob job = scheduledScaleMapper.getScheduledScaleJob(id);
        if (job == null) {
            return null;
        } else {
            return job.getScheduledTime();
        }
    }
}
```
```java
// ScheduledScaleController.java
public class ScheduledScaleController {
    public Date getScheduledTimeById(String id) {
        Long scheduleTime = scheduledScaleService.getScheduleTimeById(id));
        if (scheduleTime == null) {
            throw new JobIdNotExistException(id);
        } else {
            return new Date(scheduleTime);
        }
    }
}
```
In this example, `scheduledScaleMapper.getScheduledScaleJob(id);` returns an object of the given job. It might be null if the job with the given id not exist. 

Let's see how `Opational` can improve this code: 
The first one is using Exceptions:
```java
// ScheduledScaleService.java
public class ScheduledScaleService {
    public Opational<Long> getScheduleTimeById(String id) {
        Optional<ScaleJob> job = scheduledScaleMapper.getScheduledScaleJob(id);
        return job.map(ScaleJob::getScheduledTime);
    }
}
```
```java
// ScheduledScaleController.java
public class ScheduledScaleController {
    public Date getScheduledTimeById(String id) {
        return scheduledScaleService
                    .getScheduleTimeById(id)
                    .map(Date::new)
                    .orElseThrow(() -> new JobNotExistException(id)));
    }
}
```
The return type of `scheduledScaleMapper.getScheduledScaleJob(id);` is changed to `Opational<ScaleJob>`. The `Opational` is used in the following processes until we need to use the epoc time to construct the `Date` object. The null pointer handling is postponed to the controller who knows how to handle non-exist job better.
By using `Opational`, annoying (and easy to forget) null checking is removed. There's no need to create `Excpetion` types only for the null checking. The code becomes clearer and more readable.