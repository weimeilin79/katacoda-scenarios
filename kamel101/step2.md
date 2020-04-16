## Logging in to the Cluster via Dashboard
Writing the first Camel K application
We are going to start fresh with a simple hello world Camel K application. Opening up the Basic.java{{open}} file.
Paste the following code into the application.

<pre class="file" data-filename="Basic.java" data-target="replace">
// camel-k: language=java

import org.apache.camel.builder.RouteBuilder;

public class Basic extends RouteBuilder {
  @Override
  public void configure() throws Exception {

      from("timer:java?period=1000&fixedRate=true")
      .setHeader("example")
      .constant("java11")
      .setBody().simple("Hello World! Camel K route written in ${header.example}.")
      .to("log:ingo");

  }
}

</pre>


``kamel run Basic.java --dev``{{execute}}
