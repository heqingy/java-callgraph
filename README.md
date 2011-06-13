java-callgraph: Java Call Graph Utils
=====================================

A suite of programs for generating static and dynamic call graphs in Java.

* javacg-static: Reads classes from a jar file, walks down the method bodies and
   prints a table of caller-caller relationships.
* javacg-dynamic: Runs as a [Java agent](http://download.oracle.com/javase/6/docs/api/index.html?java/lang/instrument/package-summary.html) and instruments
  the methods of a user-defined set of classes in order to track their invocations.
  At JVM exit, prints a table of caller-callee relationships, along with a number
  of calls

#### Compile

The java-callgraph package is build with maven. Install maven and do: 

<code>
mvn install
</code>

This will produce a `target` directory with two executable jars:

- `javacg-0.1-SNAPSHOT-static.jar`: This includes the static call graph
generator
- `javacg-0.1-SNAPSHOT-dycg-agent.jar`: This includes the dynamic call
graph generator

#### Run

##### Static

<code>
java -jar javacg-0.1-SNAPSHOT-static.jar lib1.jar lib2.jar...
</code>

##### Dynamic

`javacg-dynamic` uses the
[javassist](http://www.csg.is.titech.ac.jp/~chiba/javassist/) to insert probes
at method entry and exit points. To be able to analyze a class `javassist` must
resolve all dependent classes at instrumentation time.  To do so, it reads
classes from the JVM's boot classloader. However the JVM sets the boot
classpath to use Java's default classpath implementation (`rt.jar` on
Win/Linux, `classes.jar` on the Mac).  The boot classpath can be extended using
the `-Xbootclasspath` option, which works the same as the traditional
`-classpath` option. It is advisable for `javacg-dynamic` to work as expected,
to set the boot classpath to the same, or an appropriate subset, entries as the
normal application classpath.

Moreover, since instrumenting all methods will produce huge callgraphs which
are not necessarily helpful (e.g. it will include Java's default classpath
entries), `javacg-dynamic` includes support for restricting the set of classes
to be implemented through include and exclude statements. The options are
appended to the `-javaagent` switch and has the following format

<code>-javaagent:javacg-dycg-agent.jar="incl=mylib.*,mylib2.*,java.nio.*;excl=java.nio.charset.*"</code>

The example above will instrument all classes under the the `mylib`, `mylib2` and
`java.nio` namespaces, except those that fall under the `java.nio.charset` namespace.

<code>
java -Xbootclasspath::/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:mylib.jar -javaagent:javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=mylib.*;" -classpath mylib.jar mylib.Mainclass 
</code>

#### Examples

The following examples instrument the 
[Dacapo benchmark suite](http://dacapobench.org/) to produce dynamic call graphs. 
The Dacapo benchmarks come in a single big jar archive that contains all dependency
libraries. To build the boot class path required for the javacg-dyn program, 
extract the `dacapo.jar` to a directory: all the required libraries can be found
in the `jar` directory.

Running the batik Dacapo benchmark:

<code>
java -Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:jar/batik-all.jar:jar/xml-apis-ext.jar -javaagent:target/javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=org.apache.batik.*,org.w3c.*;" -jar dacapo-9.12-bach.jar batik -s small |tail -n 10

[...]
org.apache.batik.dom.AbstractParentNode:appendChild org.apache.batik.dom.AbstractParentNode:fireDOMNodeInsertedEvent 6270
org.apache.batik.dom.AbstractParentNode:fireDOMNodeInsertedEvent org.apache.batik.dom.AbstractDocument:getEventsEnabled 6280
org.apache.batik.dom.AbstractParentNode:checkAndRemove org.apache.batik.dom.AbstractNode:getOwnerDocument 6280
org.apache.batik.dom.util.DoublyIndexedTable:put org.apache.batik.dom.util.DoublyIndexedTable$Entry:DoublyIndexedTable$Entry 6682
org.apache.batik.dom.util.DoublyIndexedTable:put org.apache.batik.dom.util.DoublyIndexedTable:hashCode 6693
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractElement:getNodeType 7198
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractDocument:getElementsByTagName 14396
org.apache.batik.dom.AbstractElement:invalidateElementsByTagName org.apache.batik.dom.AbstractDocument:getElementsByTagNameNS 28792

</code>

Running the lucene Dacapo benchmark:

<code>
java -Xbootclasspath:/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Classes/classes.jar:jar/lucene-core-2.4.jar:jar/luindex.jar -javaagent:target/javacg-0.1-SNAPSHOT-dycg-agent.jar="incl=org.apache.lucene.*;" -jar dacapo-9.12-bach.jar luindex -s small |tail -n 10

[...]
org.apache.lucene.analysis.Token:setTermBuffer org.apache.lucene.analysis.Token:growTermBuffer 43449
org.apache.lucene.analysis.CharArraySet:getSlot org.apache.lucene.analysis.CharArraySet:getHashCode 43472
org.apache.lucene.analysis.CharArraySet:getSlot org.apache.lucene.analysis.CharArraySet:equals 46107
org.apache.lucene.index.FreqProxTermsWriter:appendPostings org.apache.lucene.store.IndexOutput:writeVInt 46507
org.apache.lucene.store.IndexInput:readVInt org.apache.lucene.index.ByteSliceReader:readByte 63927
org.apache.lucene.index.TermsHashPerField:writeVInt org.apache.lucene.index.TermsHashPerField:writeByte 63927
org.apache.lucene.store.IndexOutput:writeVInt org.apache.lucene.store.BufferedIndexOutput:writeByte 94239
org.apache.lucene.index.TermsHashPerField:quickSort org.apache.lucene.index.TermsHashPerField:comparePostings 107343
org.apache.lucene.analysis.Token:termBuffer org.apache.lucene.analysis.Token:initTermBuffer 162115
org.apache.lucene.analysis.Token:termLength org.apache.lucene.analysis.Token:initTermBuffer 205554

</code>



#### Known Restrictions

* The static call graph generator does not account for methods invoked via
  reflection.

#### Author

Georgios Gousios <gousiosg@gmail.com>

#### License

[2-clause BSD](http://www.opensource.org/licenses/bsd-license.php)
