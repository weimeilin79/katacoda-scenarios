## Writing the first Camel K application
We are going to start fresh with a simple Camel K hello world application. Go to the text editor on the right, under the folder /root/camel-basic. Right click on the directory and choose new -> file and name it `Basic.java`.

Paste the following code into the application.

<pre class="file" data-filename="Basic.java" data-target="replace">
// camel-k: language=java

import org.apache.camel.builder.RouteBuilder;

public class Basic extends RouteBuilder {
  @Override
  public void configure() throws Exception {

      from("timer:java?period=1000&fixedRate=true")
      .setHeader("example")
      .constant("java")
      .setBody().simple("Hello World! Camel K route written in ${header.example}.")
      .to("log:ingo");

  }
}

</pre>

Notice you don't need to specify ANY dependency specification in the folder, Camel K will figure it out, and inject it during the build. So all you need is to JUST write your application. In this case, the Kamel binary will

``kamel run camel-basic/Basic.java --dev``{{execute}}

It's going to take 1-2 mins to start up your first application, since it needs to pull and build the image for the first time. But the next build will only take seconds.

Once it started, you will see the log printing `Hello World! Camel K route written in Java`. You can find the pod running this hello world application in the console too.

Go to  

Now, come back to the editor and try changing the word `java` to something else, such as `Java` with Capital letter. And see what happens.   
