Clarifai Java Client
====================

A simple client for the Clarifai image recognition API.
* Full API documentation can be found at: https://developer.clarifai.com.
* Sign up for a free account at: https://developer.clarifai.com/accounts/signup.
* Try the demo at: http://www.clarifai.com.


Installation
------------

Maven: add the following to the dependencies section of your pom.xml:
```
<dependency>
  <groupId>com.clarifai</groupId>
  <artifactId>clarifai-api-java</artifactId>
  <version>1.0.0</version>
</dependency>
```

Gradle: add the following to the dependencies section of your build.gradle:
```
compile "com.clarifai:clarifai-api-java:1.0.0"
```


Usage
-----
Suppose we want to recognize tags for an image on the filesystem. Here's how to do that:
```java
ClarifaiClient clarifai = new ClarifaiClient(APP_ID, APP_SECRET);
List<RecognitionResult> results =
    clarifai.recognize(new RecognitionRequest(new File("kittens.jpg")));

for (Tag tag : results.get(0).getTags()) {
  System.out.println(tag.getName() + ": " + tag.getProbability());
}
```
Each `Tag` in the `RecognitionResult` contains the name of the tag and the probability that the tag
applies to the image. Tags are drawn from a large vocabulary of English words and phrases.

The `APP_ID` and `APP_SECRET` parameters passed to the `ClarifaiClient` constructor identify the
application and can be found on the
[applications dashboard](https://developer.clarifai.com/applications/). Alternately, there is a
zero-argument constructor that reads these from the `CLARIFAI_APP_ID` and `CLARIFAI_APP_SECRET`
environment variables.

We can also pass the image as a byte array:
```java
byte[] imageBytes = ...
results = clarifai.recognize(new RecognitionRequest(imageBytes));
```

Or as a publicly-accessible URL:
```java
results = clarifai.recognize(
    new RecognitionRequest("http://www.clarifai.com/img/metro-north.jpg"));
```

You may be wondering why the `recognize` method returns a `List`. It's because recognition can
run over batches of input. For example:
```java
File[] imageFiles = {
  new File("kittens.jpg"),
  new File("puppies.png"),
  new File("cubs.gif")
};
results = clarifai.recognize(new RecognitionRequest(imageFiles));
```
This returns a `List` containing a `RecognitionResult` instance for each file.
Alternately, we could have passed byte arrays or URLs. Running recognition in batches is faster and
more efficient than sending each image individually.

How many images can we send in a batch? Let's find out:
```java
InfoResult info = clarifai.getInfo();
System.out.println(info.getMaxBatchSize());  // Prints "128"
```
The limit is currently 128 images, but may change in the future. The `InfoResult` also contains
other useful information about the API like minimum and maximum image sizes.

You may notice that occasionally, some of the tags returned by the API are wrong. In other cases,
we may be missing tags. If you or your users detect this, we encourage you to report the error back
to us. This will help us improve in the areas that you care about. The following request tells us
that the tags "kitten" and "cat" should be added to the image, and "dog" should be removed:
```java
clarifai.sendFeedback(new FeedbackRequest()
    .setDocIds(recognitionResult.getDocId())
    .setAddTags("kitten", "cat")
    .setRemoveTags("dog"));
```
The `docId` is a unique, stable ID for an image, and is returned with every `RecognitionResult`.

For more usage examples, see:
[ClarifaiClientServerTest.java](https://github.com/clarifai/clarifai-api-java/blob/master/src/test/java/com/clarifai/api/ClarifaiClientServerTest.java).


Requirements
------------
* JDK 1.6 or later


Android
-------
The client supports Android 2.3.3 (Gingerbread) and later. You'll need to add the
following permission to your `AndroidManifest.xml`:
```
<uses-permission android:name="android.permission.INTERNET" />
```

If you're using ProGuard, make sure the Clarifai API internals are not stripped out by adding the
following to your proguard.cfg:
```
-keep class com.clarifai.api.** { *; }
```
