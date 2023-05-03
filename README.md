Download Link: https://assignmentchef.com/product/solved-cs2030s-lab-6-logging
<br>
<h1></h1>

<h2>Topic Coverage</h2>

<ul>

 <li>Generics</li>

 <li>Function</li>

 <li>Lambda expression</li>

 <li><tt>map</tt> and <tt>flatMap</tt></li>

</ul>

<h2>Problem Description</h2>

In this lab, we are going to write an <i>immutable</i> <tt>Logger</tt> class to handle the context of logging changes to values while they are operated upon. In a way, the logger separates the code for logging from the main processing. Each logger wraps around a value, and everytime <tt>map</tt> or <tt>flatMap</tt> is called, a new <tt>Logger</tt> object is created. The <tt>printlog</tt> method can be invoked the output the log.

A sample logging session is shown below:

<pre><b>jshell&gt; Logger.make(5)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1)</b>$.. ==&gt; Logger[6]<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).printlog()</b>Value initialized. Value = 5Value changed! New value = 6<b>jshell&gt; Logger.make(5).map(x -&gt; x * 1)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).map(x -&gt; x * 1).printlog()</b>Value initialized. Value = 5Value unchanged. Value = 5</pre>

Notice that the log of value changes through time is output whenever the <tt>printlog()</tt> method is called. There are three different types of log messages:

<ul>

 <li>During initialization: <tt>Value initialized. Value = ..</tt></li>

 <li>When value is modified: <tt>Value changed! New value = ..</tt></li>

 <li>When value remains unchanged: <tt>Value unchanged. Value = ..</tt></li>

</ul>

<h2>Task</h2>

Your task is to write a <tt>Logger</tt> class that provides the operations <tt>make</tt>, <tt>equals</tt>, <tt>printlog</tt>, <tt>map</tt>, <tt>flatMap</tt> and <tt>test</tt>. You will also write several applications using the <tt>Logger</tt> as solutions to classic computation problems. This would allow us to look at the values changes when solving each problem.

This task is divided into several levels. Read through all the levels to see how the different levels are related.

Remember to:

<ul>

 <li>always compile your program files first before using jshell to test your program</li>

 <li>use checkstyle and javadoc comments to enhance code readability and facilitating code review</li>

</ul>




<h2>Level 1</h2>

<h2>Creating the logger</h2>

Start by writing a static method <tt>make</tt> to wrap a value within a <tt>Logger</tt>. Include the <tt>toString()</tt> method as well as the <tt>printlog()</tt> method to return the string representation of the <tt>Logger</tt> and output the log messages respectively. At this point of time, there are only initialization messages. Finally, include an <tt>equals</tt> method that returns <tt>true</tt> if the argument <tt>Logger</tt> object is the same as <tt>this</tt>, or <tt>false</tt> otherwise. Two <tt>Loggers</tt> are equal if and only if both the wrapped value as well as the logs are the same.

<pre><b>jshell&gt; Logger.make(5)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make("hello")</b>$.. ==&gt; Logger[hello]<b>jshell&gt; Logger.make("hello").printlog()</b>Value initialized. Value = hello<b>jshell&gt; Logger.make(5).equals(Logger.make(5))</b>$.. ==&gt; true<b>jshell&gt; Logger.make(5).equals(5)</b>$.. ==&gt; false<b>jshell&gt; Logger.make(5).equals(Logger.make("five"))</b>$.. ==&gt; false<b>jshell&gt; Logger.make(5).equals((Object)(Logger.make(5)))</b>$.. ==&gt; true<b>jshell&gt; /exit</b></pre>

Check the format correctness of the output by running the following on the command line:

<pre>$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test1.jsh</pre>

Check your styling by issuing the following

<pre>$ checkstyle *.java</pre>




<h2>Level 2</h2>

<h2>Creating a valid logger</h2>

We need to prevent the <tt>make</tt> method from being misused by nesting or chaining the methods, e.g.

<ul>

 <li><tt>Logger.make(Logger.make(5))</tt></li>

 <li><tt>Logger.make(5).make(7)</tt></li>

</ul>

In the case of nesting <tt>make</tt> methods, we throw an <tt>IllegalArgumentException</tt> everytime the <tt>make</tt> method takes in another <tt>Logger</tt> as argument.

To prevent chaining, we can make the <tt>Logger</tt> an interface with the static method <tt>make</tt>, and let <tt>LoggerImpl</tt> be the concrete class that implements the functionalities of logging. Here, we restrict the chaining of <tt>make</tt> method calls at the expense of introducing a cyclic dependency between <tt>Logger</tt> and <tt>LoggerImpl</tt>… <i>that is, unless someone comes up with a better idea…</i>

In addition, passing a <tt>null</tt> to <tt>make</tt> will also result in an <tt>IllegalArgumentException</tt> thrown.

<pre><b>jshell&gt; Logger.make(5)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5) instanceof Logger </b>$.. ==&gt; true<b>jshell&gt; try { Logger.make(Logger.make(7)); } catch (Exception e) { System.out.println(e); }</b>java.lang.IllegalArgumentException: already a Logger<b>jshell&gt; Logger.make(5).make(7)</b>|  Error:|  illegal static interface method call|    the receiver expression should be replaced with the type qualifier 'Logger&lt;java.lang.Integer&gt;'|  Logger.make(5).make(7)|  ^--------------------^<b>jshell&gt; try { Logger.make(null); } catch (Exception e) { System.out.println(e); }</b>java.lang.IllegalArgumentException: argument cannot be null<b>jshell&gt; /exit</b></pre>

Check the format correctness of the output by running the following on the command line:

<pre>$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test2.jsh</pre>

Check your styling by issuing the following

<pre>$ checkstyle *.java</pre>

<h2>Level 3</h2>

<h2>The <tt>map</tt> method</h2>

Include a <tt>map</tt> method that takes in a function of the form <tt>Function&lt;? super T, ? extends U&gt;</tt>, applies it on the value, and wraps the result in a <tt>Logger</tt>.

<pre><b>jshell&gt; Logger.make(5)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1)</b>$.. ==&gt; Logger[6]<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).printlog()</b>Value initialized. Value = 5Value changed! New value = 6<b>jshell&gt; Logger&lt;Integer&gt; a = Logger.make(5)</b><b>jshell&gt; a.printlog()</b>Value initialized. Value = 5<b>jshell&gt; a.map(x -&gt; x + 1)</b>$.. ==&gt; Logger[6]<b>jshell&gt; a.printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make(5).map(x -&gt; x)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).map(x -&gt; x).printlog()</b>Value initialized. Value = 5Value unchanged. Value = 5<b>jshell&gt; Logger.make(5).equals(Logger.make(5).map(x -&gt; x))</b>$.. ==&gt; false<b>jshell&gt; Logger.make(5).map(x -&gt; x).equals(Logger.make(5))</b>$.. ==&gt; false<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x - 1)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x - 1).printlog()</b>Value initialized. Value = 5Value changed! New value = 6Value changed! New value = 5<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).map(x -&gt; x - 1).equals(Logger.make(5))</b>$.. ==&gt; false<b>jshell&gt; Logger.make("hello").map(String::length)</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make("hello").map(String::length).printlog()</b>Value initialized. Value = helloValue changed! New value = 5<b>jshell&gt; Function&lt;Object, Boolean&gt; f = x -&gt; x.equals(x)</b><b>jshell&gt; Logger.make("hello").map(f)</b>$.. ==&gt; Logger[true]<b>jshell&gt; Function&lt;String, Number&gt; g = x -&gt; x.length();</b><b>jshell&gt; Logger&lt;Number&gt; lognum = Logger.make("hello").map(g)</b><b>jshell&gt; lognum</b>lognum ==&gt; Logger[5]<b>jshell&gt; /exit</b></pre>

Check the format correctness of the output by running the following on the command line:

<pre>$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test3.jsh</pre>

Check your styling by issuing the following

<pre>$ checkstyle *.java</pre>

<h2>Level 4</h2>

<h2>The <tt>flatMap</tt> method</h2>

We are now ready to write the <tt>flatMap</tt> method that takes a function of the form <tt>Function&lt;? super T, ? extends Logger&lt;? extends U&gt;&gt;</tt>. Pay special attention to how the logs are to be combined.

<pre><b>jshell&gt; Logger.make(5).printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x))</b>$.. ==&gt; Logger[5]<b>jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x)).printlog()</b>Value initialized. Value = 5<b>jshell&gt; Logger.make(5).flatMap(x -&gt; Logger.make(x)).equals(Logger.make(5))</b>$.. ==&gt; true<b>jshell&gt; Logger&lt;Integer&gt; a = Logger.make(5).flatMap(x -&gt; Logger.make(x).map(y -&gt; y + 2)).flatMap(y -&gt; Logger.make(y).map(z -&gt; z * 10))</b><b>jshell&gt; Logger&lt;Integer&gt; b = Logger.make(5).flatMap(x -&gt; Logger.make(x).map(y -&gt; y + 2).flatMap(y -&gt; Logger.make(y).map(z -&gt; z * 10)))</b><b>jshell&gt; a.printlog()</b>Value initialized. Value = 5Value changed! New value = 7Value changed! New value = 70<b>jshell&gt; b.printlog()</b>Value initialized. Value = 5Value changed! New value = 7Value changed! New value = 70<b>jshell&gt; a.equals(b)</b>$.. ==&gt; true<b>jshell&gt; Logger&lt;Integer&gt; c = Logger.make(5).map(x -&gt; x + 2).map(x -&gt; x * 10)</b><b>jshell&gt; a.equals(c)</b>$.. ==&gt; true<b>jshell&gt; Function&lt;Object, Logger&lt;Boolean&gt;&gt; f = x -&gt; Logger.make(x).map(y -&gt; y.equals(y))</b><b>jshell&gt; Logger.make("hello").flatMap(f)</b>$.. ==&gt; Logger[true]<b>jshell&gt; Function&lt;String, Logger&lt;Number&gt;&gt; g = x -&gt; Logger.make(x).map(y -&gt; y.length())</b><b>jshell&gt; Logger&lt;Number&gt; lognum = Logger.make("hello").flatMap(g)</b><b>jshell&gt; lognum</b>lognum ==&gt; Logger[5]<b>jshell&gt; /exit</b></pre>

Check the format correctness of the output by running the following on the command line:

<pre>$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test4.jsh</pre>

Check your styling by issuing the following

<pre>$ checkstyle *.java</pre>

<h2>Level 5</h2>

Let’s write some applications using JShell that makes use of our <tt>Logger</tt> so as to observe how the values changes over the course computation. Save your methods in the file <tt>level5.jsh</tt>.

Define an <tt>add(Logger&lt;Integer&gt; a, int b)</tt> method that returns the result of <tt>a</tt> added to <tt>b</tt> wrapped in a <tt>Logger</tt> that preserves the log of all operations of <tt>a</tt>, as well as the addition to <tt>b</tt>.

<pre><b>jshell&gt; add(Logger.make(5), 6)</b>$.. ==&gt; Logger[11]<b>jshell&gt; add(Logger.make(5), 6).printlog()</b>Value initialized. Value = 5Value changed! New value = 11<b>jshell&gt; add(Logger.make(5).map(x -&gt; x * 2), 6)</b>$.. ==&gt; Logger[16]<b>jshell&gt; add(Logger.make(5).map(x -&gt; x * 2), 6).printlog()</b>Value initialized. Value = 5Value changed! New value = 10Value changed! New value = 16</pre>

The sum of non-negative integers from <tt>0</tt> to <tt>n</tt> (inclusive of both) can be computed <i>recursively</i> using

<pre>int sum(int n) {    if (n == 0) {        return 0;    } else {        return sum(n - 1) + n;    }}</pre>

Redefine the above method such that it returns the result wrapped in a <tt>Logger</tt>. You may find the above <tt>add</tt> method useful.

<pre><b>jshell&gt; sum(0)</b>$.. ==&gt; Logger[0]<b>jshell&gt; sum(0).printlog()</b>Value initialized. Value = 0<b>jshell&gt; sum(5)</b>$.. ==&gt; Logger[15]<b>jshell&gt; sum(5).printlog()</b>Value initialized. Value = 0Value changed! New value = 1Value changed! New value = 3Value changed! New value = 6Value changed! New value = 10Value changed! New value = 15</pre>

The <i>Collatz conjecture</i> (or the <tt>3n+1</tt> <i>Conjecture</i>) is a process of generating a sequence of numbers starting with a positive integer that <i>seems to always</i> end with <tt>1</tt>.

<pre>int f(int n) {   if (n == 1) {      return 1;   } else if (n % 2 == 0) {      return f(n / 2);   } else {      return f(3 * n + 1);   } }</pre>

Write the function <tt>f</tt> that takes in <tt>n</tt> and returns a <tt>Logger&lt;Integer&gt;</tt> that wraps around the final value, with a log of the value changes over time. You should include a <tt>test</tt> method in the <tt>Logger</tt> that takes in a <tt>Predicate</tt> and returns <tt>true</tt> if the value within the <tt>Logger</tt> satisfies the predicate, and <tt>false</tt> otherwise.

<pre><b>jshell&gt; Logger.make(5).test(x -&gt; x == 5)</b>$.. ==&gt; true<b>jshell&gt; Logger.make(5).map(x -&gt; x + 1).test(x -&gt; x % 2 != 0)</b>$.. ==&gt; false<b>jshell&gt; Logger.make("hello").test(x -&gt; x.length() == 5)</b>$.. ==&gt; true<b>jshell&gt; f(16)</b>$.. ==&gt; Logger[1]<b>jshell&gt; f(16).printlog()</b>Value initialized. Value = 16Value changed! New value = 8Value changed! New value = 4Value changed! New value = 2Value changed! New value = 1<b>jshell&gt; f(10)</b>$.. ==&gt; Logger[1]<b>jshell&gt; f(10).printlog()</b>Value initialized. Value = 10Value changed! New value = 5Value changed! New value = 15Value changed! New value = 16Value changed! New value = 8Value changed! New value = 4Value changed! New value = 2Value changed! New value = 1</pre>

Check the format correctness of the output by running the following on the command line:

<pre>$ javac -Xlint:rawtypes *.java$ jshell -q your_java_files_in_bottom-up_dependency_order &lt; test5.jsh</pre>

Check your styling by issuing the following

<pre>$ checkstyle *.java</pre>