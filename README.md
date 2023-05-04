Download Link: https://assignmentchef.com/product/solved-cse-344-homework-5-database-application-and-transaction-management
<br>
<strong>Objectives:</strong> To gain experience with database application development and transaction management. To learn how to use SQL from within Java via JDBC.

<strong>Assignment tools:</strong>

<ul>

 <li><a href="https://www.microsoft.com/sqlserver" rel="nofollow">SQL Server</a> through <a href="https://azure.microsoft.com/en-us/services/sql-database/" rel="nofollow">SQL Azure</a></li>

 <li>Maven (if using OSX, we recommend using Homebrew and installing with <code>brew install maven</code>)</li>

 <li><a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/PreparedStatement.html" rel="nofollow">Prepared Statements</a></li>

 <li>starter code files</li>

</ul>

<h2>Assignment Details</h2>

<strong>Read through this whole section before starting this project.</strong> There is a lot of valuable information here, and some implementation details depend on others.

Congratulations, you are opening your own flight booking service!

In this homework, you have two main tasks:

<ul>

 <li>Design a database of your customers and the flights they book</li>

 <li>Complete a working prototype of your flight booking application that connects to the database (in Azure) then allows customers to use a CLI to search, book, cancel, etc. flights</li>

</ul>

You will also be writing a few test cases and explaining your design in a short writeup. We have already provided code for a UI (<code>FlightService.java</code>) and partial backend (<code>Query.java</code>). For this homework, your task is to implement the rest of the backend. In real life, you would develop a web-based interface instead of a CLI, but we use a CLI to simplify this homework.

For this lab, you can use any of the classes from the <a href="https://docs.oracle.com/en/java/javase/11/docs/api/index.html" rel="nofollow">Java 11 standard JDK</a>.

<h4><a id="user-content-connect-your-application-to-your-database" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#connect-your-application-to-your-database" aria-hidden="true"></a>Connect your application to your database</h4>

You will need to access your Flights database on SQL Azure from HW3. Alternatively, you may create a new database and define the Flights data as follows.

<h5><a id="user-content-connect-to-our-external-flights-tables" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#connect-to-our-external-flights-tables" aria-hidden="true"></a>Connect to our external Flights tables</h5>

To improve performance and make sure you don’t alter the flights data, reference the Flights tables as <code>EXTERNAL TABLE</code>s. Every query to the Flights tables from your database will be redirected to a faster class server.

<ul>

 <li>Step 1: Drop all your table:</li>

 <li>Step 2: Create a credential, please enter exactly like this:</li>

 <li>Step 3: Create external data source (it’s one of our TA’s server, don’t judge him):</li>

 <li>Step 4: Create the external tables:</li>

 <li>Step 5: Check that you can query the external tables:</li>

</ul>

<h5><a id="user-content-configure-your-jdbc-connection" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#configure-your-jdbc-connection" aria-hidden="true"></a>Configure your JDBC Connection</h5>

You need to configure the appropriate information to connect <code>Query.java</code> to your database.

In the top level directory, create a file named <code>dbconn.properties</code> with the following contents:

<pre><code># Database connection settings# TODO: Enter the server URL.hw5.server_url = SERVER_URL# TODO: Enter your database name.hw5.database_name = DATABASE_NAME# TODO: Enter the admin username of your server.hw5.username = USERNAME# TODO: Add your admin password.hw5.password = PASSWORD</code></pre>

Check your Azure server and fill in the connection details:

<ul>

 <li>The server URL will be of the form <code>[your_server_name].database.windows.net</code></li>

 <li>The database name, admin, and password will be whatever you specified</li>

 <li>If the connection isn’t working for some reason, try using the fully qualified username: <code>hw5.username = <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="2b7e786e7974656a666e6b786e797d6e79">[email protected]</a>_NAME</code></li>

</ul>

<h4><a id="user-content-build-the-application" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#build-the-application" aria-hidden="true"></a>Build the application</h4>

Make sure your application can run by entering the following commands in the directory of the starter code and <code>pom.xml</code> file. This first command will package the application files and any dependencies into a single .jar file:

This second command will run the main method from <code>FlightService.java</code>, the interface logic for what you will implement in <code>Query.java</code>:

If you want to run directly without creating the jar, you can run:

If you get our UI below, you are good to go for the rest of the lab!

<pre><code>*** Please enter one of the following commands ***&gt; create &lt;username&gt; &lt;password&gt; &lt;initial amount&gt;&gt; login &lt;username&gt; &lt;password&gt;&gt; search &lt;origin city&gt; &lt;destination city&gt; &lt;direct&gt; &lt;day&gt; &lt;num itineraries&gt;&gt; book &lt;itinerary id&gt;&gt; pay &lt;reservation id&gt;&gt; reservations&gt; cancel &lt;reservation id&gt;&gt; quit</code></pre>

<h4><a id="user-content-data-model" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#data-model" aria-hidden="true"></a>Data Model</h4>

The flight service system consists of the following logical entities. These entities are <em>not necessarily database tables</em>. It is up to you to decide what entities to store persistently and create a physical schema design that has the ability to run the operations below, which make use of these entities.

<ul>

 <li><strong>Flights / Carriers / Months / Weekdays</strong>: modeled the same way as HW3.For this application, we have very limited functionality so you shouldn’t need to modify the schema from HW3 nor add any new table to reason about the data.</li>

 <li><strong>Users</strong>: A user has a username (<code>varchar</code>), password (<code>varbinary</code>), and balance (<code>int</code>) in their account. All usernames should be unique in the system. Each user can have any number of reservations. Usernames are case insensitive (this is the default for SQL Server). Since we are salting and hashing our passwords through the Java application, passwords are case sensitive. You can assume that all usernames and passwords have at most 20 characters.</li>

 <li><strong>Itineraries</strong>: An itinerary is either a direct flight (consisting of one flight: origin –&gt; destination) or a one-hop flight (consisting of two flights: origin –&gt; stopover city, stopover city –&gt; destination). Itineraries are returned by the search command.</li>

 <li><strong>Reservations</strong>: A booking for an itinerary, which may consist of one (direct) or two (one-hop) flights. Each reservation can either be paid or unpaid, cancelled or not, and has a unique ID.</li>

</ul>

Create other tables or indexes you need for this assignment in <code>createTables.sql</code> (see below).

<h4><a id="user-content-requirements" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#requirements" aria-hidden="true"></a>Requirements</h4>

The following are the functional specifications for the flight service system, to be implemented in <code>Query.java</code> (see the method stubs in the starter code for full specification as to what error message to return, etc):

<ul>

 <li><strong>create</strong> takes in a new username, password, and initial account balance as input. It creates a new user account with the initial balance. It should return an error if negative, or if the username already exists. Usernames and passwords are checked case-insensitively. You can assume that all usernames and passwords have at most 20 characters. We will store the salted password hash and the salt itself to avoid storing passwords in plain text. Use the following code snippet to as a template for computing the hash given a password string:</li>

 <li><strong>login</strong> takes in a username and password, and checks that the user exists in the database and that the password matches. To compute the hash, adapt the above code. Within a single session (that is, a single instance of your program), only one user should be logged in. You can track this via a local variable in your program. If a second login attempt is made, please return “User already logged in”. Across multiple sessions (that is, if you run your program multiple times), the same user is allowed to be logged in. This means that you do not need to track a user’s login status inside the database.</li>

 <li><strong>search</strong> takes as input an origin city (string), a destination city (string), a flag for only direct flights or not (0 or 1), the date (int), and the maximum number of itineraries to be returned (int). For the date, we only need the day of the month, since our dataset comes from July 2015. Return only flights that are not canceled, ignoring the capacity and number of seats available. If the user requests n itineraries to be returned, there are a number of possibilities:

  <ul>

   <li>direct=1: return up to n direct itineraries</li>

   <li>direct=0: return up to n direct itineraries. If there are only k direct itineraries (where k &lt; n), then return the k direct itineraries and up to (n-k) of the shortest indirect itineraries with the flight times. For one-hop flights, different carriers can be used for the flights. For the purpose of this assignment, an indirect itinerary means the first and second flight only must be on the same date (i.e., if flight 1 runs on the 3rd day of July, flight 2 runs on the 4th day of July, then you can’t put these two flights in the same itinerary as they are not on the same day).</li>

  </ul>Sort your results. In all cases, the returned results should be primarily sorted on total actual_time (ascending). If a tie occurs, break that tie by the fid value. Use the first then the second fid for tie-breaking.Below is an example of a single direct flight from Seattle to Boston. Actual itinerary numbers might differ, notice that only the day is printed out since we assume all flights happen in July 2015:<pre><code>Itinerary 0: 1 flight(s), 297 minutesID: 60454 Day: 1 Carrier: AS Number: 24 Origin: Seattle WA Dest: Boston MA Duration: 297 Capacity: 14 Price: 140</code></pre>Below is an example of two indirect flights from Seattle to Boston:<pre><code>Itinerary 0: 2 flight(s), 317 minutesID: 704749 Day: 10 Carrier: AS Number: 16 Origin: Seattle WA Dest: Orlando FL Duration: 159 Capacity: 10 Price: 494ID: 726309 Day: 10 Carrier: B6 Number: 152 Origin: Orlando FL Dest: Boston MA Duration: 158 Capacity: 0 Price: 104Itinerary 1: 2 flight(s), 317 minutesID: 704749 Day: 10 Carrier: AS Number: 16 Origin: Seattle WA Dest: Orlando FL Duration: 159 Capacity: 10 Price: 494ID: 726464 Day: 10 Carrier: B6 Number: 452 Origin: Orlando FL Dest: Boston MA Duration: 158 Capacity: 7 Price: 760</code></pre>Note that for one-hop flights, the results are printed in the order of the itinerary, starting from the flight leaving the origin and ending with the flight arriving at the destination.The returned itineraries should start from 0 and increase by 1 up to n as shown above. If no itineraries match the search query, the system should return an informative error message. See <code>Query.java</code> for the actual text.The user need not be logged in to search for flights.All flights in an indirect itinerary should be under the same itinerary ID. In other words, the user should only need to book once with the itinerary ID for direct or indirect trips.</li>

 <li><strong>book</strong> lets a user book an itinerary by providing the itinerary number as returned by a previous search. The user must be logged in to book an itinerary, and must enter a valid itinerary id that was returned in the last search that was performed <em>within the same login session</em>. Make sure you make the corresponding changes to the tables in case of a successful booking. Once the user logs out (by quitting the application), logs in (if they previously were not logged in), or performs another search within the same login session, then all previously returned itineraries are invalidated and cannot be booked.A user cannot book a flight if the flight’s maximum capacity would be exceeded. Each flight’s capacity is stored in the Flights table as in HW3, and you should have records as to how many seats remain on each flight based on the reservations.If booking is successful, then assign a new reservation ID to the booked itinerary. Note that 1) each reservation can contain up to 2 flights (in the case of indirect flights), and 2) each reservation should have a unique ID that incrementally increases by 1 for each successful booking.</li>

 <li><strong>pay</strong> allows a user to pay for an existing unpaid reservation. It first checks whether the user has enough money to pay for all the flights in the given reservation. If successful, it updates the reservation to be paid.</li>

 <li><strong>reservations</strong> lists all reservations for the currently logged-in user. Each reservation must have <em><strong>a unique identifier (which is different for each itinerary) in the entire system</strong></em>, starting from 1 and increasing by 1 after each reservation is made.There are many ways to implement this. One possibility is to define a “ID” table that stores the next ID to use, and update it each time when a new reservation is made successfully.The user must be logged in to view reservations. The itineraries should be displayed using similar format as that used to display the search results, and they should be shown in increasing order of reservation ID under that username. Cancelled reservations should not be displayed.</li>

 <li><strong>cancel</strong> lets a user to cancel an existing uncanceled reservation. The user must be logged in to cancel reservations and must provide a valid reservation ID. Make sure you make the corresponding changes to the tables in case of a successful cancellation (e.g., if a reservation is already paid, then the customer should be refunded).</li>

 <li><strong>quit</strong> leaves the interactive system and logs out the current user (if logged in).</li>

</ul>

Refer to the Javadoc in <code>Query.java</code> for full specification and the expected responses of the commands above.

<em><strong>CAUTION:</strong></em> Make sure your code produces outputs in the same formats as prescribed! (see test cases and Javadoc for what to expect)

<h4><a id="user-content-testing" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#testing" aria-hidden="true"></a>Testing:</h4>

To test that your application works correctly, we have provided a test harness using the JUnit framework. Our test harness will compile your code and run all the test cases in the provided <code>cases/</code> folder. To run the harness, execute in the project directory:

If you want to run a single test file or run files from a different directory (recursively), you can run the following command:

For every test case it will either print pass or fail, and for all failed cases it will dump out what the implementation returned, and you can compare it with the expected output in the corresponding case file.

Each test case file is of the following format:

While we’ve provided test cases for most of the methods, the testing we provide is partial (although significant). It is <strong>up to you</strong> to implement your solutions so that they completely follow the provided specification.

For this homework, you’re required to write test cases for each of the commands (you don’t need to test <code>quit</code>). Separate each test case in its own file and name it <code>&lt;command name&gt;_&lt;some descriptive name for the test case&gt;.txt</code> and turn them in. It’s fine to turn in test cases for erroneous conditions (e.g., booking on a full flight, logging in with a non-existent username).

<h2><a id="user-content-milestone-1" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#milestone-1" aria-hidden="true"></a>Milestone 1:</h2>

<h4><a id="user-content-database-design" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#database-design" aria-hidden="true"></a>Database design</h4>

Your first task is to design and add tables to your flights database. You should decide on the relational tables given the logical data model described above. Feel free to use your E/R diagram from HW4 as a starting point. You may discover you need to adjust your design over time as you discover new requirements. You can add other tables to your database as well.

You should fill the provided <code>createTables.sql</code> file with <code>CREATE TABLE</code> and any <code>INSERT</code> statements (and optionally any <code>CREATE INDEX</code> statements) needed to implement the logical data model above. We will test your implementation with the flights table populated with HW3 data using the schema above, and then running your <code>createTables.sql</code>. So make sure your file is runnable on SQL Azure through SQL Server Management Studio or the Azure web interface.

Please note that due to the nature of <code>EXTERNAL TABLE</code>, you will not be able to create foreign keys that reference flights (<code>FOREIGN KEY REFERENCE Flights</code>) or <code>INDEX</code>es over the Flights table. For those foreign keys, just make them normal columns in your tables. You also do not need to keep <code>CREATE EXTERNAL TABLE</code> statements in this file.

<em>NOTE:</em> You may want to write a separate script file with <code>DROP TABLE</code> or <code>DELETE FROM</code> statements; it’s useful to run it whenever you find a bug in your schema or data. You don’t need to turn in anything for this.

<h4><a id="user-content-java-customer-application" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#java-customer-application" aria-hidden="true"></a>Java customer application</h4>

Your second task is to start writing the Java application that your customers will use. To make your life easier, we’ve broken down this process into 5 different steps across the two milestones (see details below). You only need to modify <code>Query.java</code>. Do not modify <code>FlightService.java</code>.

We expect that you use <a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/PreparedStatement.html" rel="nofollow">Prepared Statements</a> where applicable. Please make your code reasonably easy to read.

To keep things neat we have provided you with the <code>Flight</code> inner class that acts as a container for your flight data. The <code>toString</code> method in the Flight class matches what is needed in methods like <code>search</code>. We have also provided a sample helper method <code>checkFlightCapacity</code> that uses a prepared statement. <code>checkFlightCapacity</code> outlines the way we think forming prepared statements should go for this assignment (creating a constant SQL string, preparing it in the prepareStatements method, and then finally using it).

<h4><a id="user-content-step-1-implement-cleartables" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#step-1-implement-cleartables" aria-hidden="true"></a>Step 1: Implement clearTables</h4>

Implement the <code>clearTables</code> method in <code>Query.java</code> to clear the contents of any tables you have created for this assignment (e.g., reservations). However, do NOT drop any of them and do NOT modify the contents or drop the <code>Flights</code> table. <strong>Any attempt to modify the <code>Flights</code> table will result in a harsh penalty in your score of this homework.</strong>

After calling this method the database should be in the same state as the beginning, i.e., with the flights table populated and <code>createTables.sql</code> called. This method is for running the test harness where each test case is assumed to start with a clean database. You will see how this works after running the test harness.

<strong><code>clearTables</code> should not take more than a minute.</strong> Make sure your database schema is designed with this in mind.

<h4><a id="user-content-step-2-implement-create-login-and-search" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#step-2-implement-create-login-and-search" aria-hidden="true"></a>Step 2: Implement create, login, and search</h4>

Implement the <code>create</code>, <code>login</code> and <code>search</code> commands in <code>Query.java</code>. Using <code>mvn test</code>, you should now pass our provided test cases that only involve these three commands.

After implementation of these class, you should pass the following test:

<h4><a id="user-content-step-3-write-some-test-cases" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#step-3-write-some-test-cases" aria-hidden="true"></a>Step 3: Write some test cases</h4>

Write at least 1 test case for each of the three commands you just implemented. Follow the <a href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#testing">same format</a> as our provided test cases. Include your written test files in the provided <code>cases/mycases/</code> folder together with our provided test files.

Using <code>mvn test -Dtest.casess="cases/mycases"</code>, you should now also pass your newly created test cases.

<h4><a id="user-content-what-to-turn-in" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#what-to-turn-in" aria-hidden="true"></a>What to turn in:</h4>

<ul>

 <li>Customer database schema in <code>createTables.sql</code></li>

 <li>Your <code>Query.java</code> with the <code>create</code>, <code>login</code> and <code>search</code> commands implemented</li>

 <li>At least 3 custom test cases (one for each command) in the <code>cases/mycases/</code> folder, with a descriptive name for each case</li>

</ul>

<h4><a id="user-content-grading" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#grading" aria-hidden="true"></a>Grading:</h4>

This milestone is worth 16 points (about 10% of the total homework grade) based on whether your create, login, and search methods pass the provided test cases.

<h2><a id="user-content-milestone-2" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#milestone-2" aria-hidden="true"></a>Milestone 2:</h2>

<h4><a id="user-content-step-4-implement-book-pay-reservations-cancel-and-add-transactions" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#step-4-implement-book-pay-reservations-cancel-and-add-transactions" aria-hidden="true"></a>Step 4: Implement book, pay, reservations, cancel, and add transactions!</h4>

Implement the <code>book</code>, <code>pay</code> , <code>reservations</code> and <code>cancel</code> commands in <code>Query.java</code>.

While implementing &amp; trying out these commands, you’ll notice that there are problems when multiple users try to use your service concurrently. To resolve this challenge, you will need to implement transactions that ensure concurrent commands do not conflict.

Think carefully as to <em>which</em> commands need transaction handling. Do the <code>create</code>, <code>login</code> and <code>search</code> commands need transaction handling? Why or why not?

Or you can run all non transaction test:

<h5><a id="user-content-transaction-management" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#transaction-management" aria-hidden="true"></a>Transaction management</h5>

You must use SQL transactions to guarantee ACID properties: we have set the isolation level for your <code>Connection</code>, and you need to define <code>begin-transaction</code> and <code>end-transaction</code> statements and insert them in appropriate places in <code>Query.java</code>. In particular, you must ensure that the following constraints are always satisfied, even if multiple instances of your application talk to the database at the same time:

<em>C1:</em> Each flight should have a maximum capacity that must not be exceeded. Each flight’s capacity is stored in the Flights table as in HW3, and you should have records as to how many seats remain on each flight based on the reservations.

<em>C2:</em> A customer may have at most one reservation on any given day, but they can be on more than 1 flight on the same day. (i.e., a customer can have one reservation on a given day that includes two flights, because the reservation is for a one-hop itinerary).

You must use transactions correctly such that race conditions introduced by concurrent execution cannot lead to an inconsistent state of the database. For example, multiple customers may try to book the same flight at the same time. Your properly designed transactions should prevent that.

Design transactions correctly. Avoid including user interaction inside a SQL transaction: that is, don’t begin a transaction then wait for the user to decide what she wants to do (why?). The rule of thumb is that transactions need to be <em>as short as possible, but not shorter</em>.

Your <code>executeQuery</code> call will throw a <code>SQLException</code> when an error occurs (e.g., multiple customers try to book the same flight concurrently). Make sure you handle the <code>SQLException</code> appropriately. For instance, if a seat is still available, the booking should eventually go through (even though you might need to retry due to <code>SQLException</code>s being thrown). If no seat is available, the booking should be rolled back, etc.

When one uses a DBMS, recall that by default <strong>each statement executes in its own transaction</strong>. As discussed in lecture, to group multiple statements into a transaction, we use:

This is the same when executing transactions from Java: by default, each SQL statement will be executed as its own transaction. To group multiple statements into one transaction in Java, you can do one of these approaches:

<em>Approach 1</em>:

Execute the SQL code for <code>BEGIN TRANSACTION</code> and friends directly, using the SQL code below (also check out SQL Azure’s <a href="https://docs.microsoft.com/en-us/sql/t-sql/language-elements/transactions-transact-sql?view=sql-server-ver15" rel="nofollow">transactions documentation</a>):

Please do not use <code>PreparedStatement</code> for these query, since SQLServer JDBC update, PreparedStatement must fully commit to execute (i.e. have the same number of <code>BEGIN</code> and <code>COMMIT</code>). Second style below is preferred.

<em>Approach 2</em>:

When auto-commit is set to true, each statement executes in its own transaction. With auto-commit set to false, you can execute many statements within a single transaction. By default, on any new connection to a DB auto-commit is set to true.

The total amount of code to add transaction handling is in fact small, but getting everything to work harmoniously may take some time. Debugging transactions can be a pain, but print statements are your friend!

Now you should pass all the provided test in cases during <code>mvn test</code>

<h4><a id="user-content-step-5-write-more-transaction-test-cases" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#step-5-write-more-transaction-test-cases" aria-hidden="true"></a>Step 5: Write more (transaction) test cases</h4>

Write at least 1 test case for each of the four commands you just implemented. Follow the <a href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#testing">same format</a> as our provided test cases.

In addition, write at least 1 <em>parallel</em> test case for each of the 7 commands. By <em>parallel</em>, we mean concurrent users interfacing with your database, with each user in a seperate application instance.

Remember that each test case file is in the following format:

The <code>*</code> separates between commands and the expected output. To test with multiple concurrent users, simply add more <code>[command...] * [expected output...]</code> pairs to the file, for instance:

Each user is expected to start concurrently in the beginning. If there are multiple output possibilities due to transactional behavior, then separate each group of expected output with <code>|</code>. See <code>book_2UsersSameFlight.txt</code> for an example.

Put your written test files in the <code>cases/mycases/</code> folder.

Using <code>mvn test -Dtest.cases="cases"</code>, you should now also pass ALL the test cases in the <code>cases</code> folder – it will recursively run the provided test cases as well as your own.

<em>Congratulations!</em> You now finish the entire flight booking application and is ready to launch your flight booking business &#x1f642;

<h4><a id="user-content-write-down-your-design" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#write-down-your-design" aria-hidden="true"></a>Write down your design</h4>

Please describe and/or draw your database design. This is so we can understand your implementation as close to what you were thinking. Explain your design choices in creating new tables. Also, describe your thought process in deciding what needs to be persisted on the database and what can be implemented in-memory (not persisted on the database). Please be concise in your writeup (&lt; half a page).

Save this file in <code>writeup.md</code> in the same folder of <code>createTables.sql</code>. You can add images to markdown by using <code>![imagename](./relative/path/to/image.png)</code>. Make sure any images is also pushed to git.

<h4><a id="user-content-what-to-turn-in-1" class="anchor" href="https://github.com/theoliao1998/Data-Management/blob/master/5%20Database%20Application%20and%20Transaction%20Management/hw5.md#what-to-turn-in-1" aria-hidden="true"></a>What to turn in:</h4>

<ul>

 <li>Customer database schema in <code>createTables.sql</code></li>

 <li>Your fully-completed version of the <code>Query.java</code></li>

 <li>At least 14 custom test cases (one normal &amp; one parallel for each command) in the <code>cases/mycases</code> folder, with a descriptive name for each case</li>

 <li>A <code>writeup.md</code> and any images it requires</li>

</ul>

5/5 - (1 vote)

<pre><span class="pl-k">DROP</span> <span class="pl-k">TABLE</span> IF EXISTS Flights;<span class="pl-k">DROP</span> <span class="pl-k">TABLE</span> IF EXISTS Carriers;<span class="pl-k">DROP</span> <span class="pl-k">TABLE</span> IF EXISTS Weekdays;<span class="pl-k">DROP</span> <span class="pl-k">TABLE</span> IF EXISTS Months;</pre>

<pre>CREATE MASTER KEY ENCRYPTION BY PASSWORD <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">'</span>CsE344RaNdOm$Key<span class="pl-pds">'</span></span>;<span class="pl-k">CREATE</span> <span class="pl-k">DATABASE</span> <span class="pl-en">SCOPED</span>CREDENTIAL QueryCredentialWITH IDENTITY <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">'</span>reader2020<span class="pl-pds">'</span></span>, SECRET <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">'</span>20wiUWcse()<span class="pl-pds">'</span></span>;</pre>

<pre>CREATE EXTERNAL DATA SOURCE CSE344_EXTERNALWITH( TYPE <span class="pl-k">=</span> RDBMS,  LOCATION<span class="pl-k">=</span><span class="pl-s"><span class="pl-pds">'</span>khangishandsome.database.windows.net<span class="pl-pds">'</span></span>,  DATABASE_NAME <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">'</span>cse344_readonly<span class="pl-pds">'</span></span>,  CREDENTIAL <span class="pl-k">=</span> QueryCredential);</pre>

<pre>CREATE EXTERNAL TABLE Flights(  fid <span class="pl-k">int</span>,  month_id <span class="pl-k">int</span>,  day_of_month <span class="pl-k">int</span>,  day_of_week_id <span class="pl-k">int</span>,  carrier_id <span class="pl-k">varchar</span>(<span class="pl-c1">7</span>),  flight_num <span class="pl-k">int</span>,  origin_city <span class="pl-k">varchar</span>(<span class="pl-c1">34</span>),  origin_state <span class="pl-k">varchar</span>(<span class="pl-c1">47</span>),  dest_city <span class="pl-k">varchar</span>(<span class="pl-c1">34</span>),  dest_state <span class="pl-k">varchar</span>(<span class="pl-c1">46</span>),  departure_delay <span class="pl-k">int</span>,  taxi_out <span class="pl-k">int</span>,  arrival_delay <span class="pl-k">int</span>,  canceled <span class="pl-k">int</span>,  actual_time <span class="pl-k">int</span>,  distance <span class="pl-k">int</span>,  capacity <span class="pl-k">int</span>,  price <span class="pl-k">int</span>) WITH (DATA_SOURCE <span class="pl-k">=</span> CSE344_EXTERNAL);CREATE EXTERNAL TABLE Carriers(  cid <span class="pl-k">varchar</span>(<span class="pl-c1">7</span>),  name <span class="pl-k">varchar</span>(<span class="pl-c1">83</span>)) WITH (DATA_SOURCE <span class="pl-k">=</span> CSE344_EXTERNAL);CREATE EXTERNAL TABLE Weekdays(  did <span class="pl-k">int</span>,  day_of_week <span class="pl-k">varchar</span>(<span class="pl-c1">9</span>)) WITH (DATA_SOURCE <span class="pl-k">=</span> CSE344_EXTERNAL);CREATE EXTERNAL TABLE Months(  mid <span class="pl-k">int</span>,  month <span class="pl-k">varchar</span>(<span class="pl-c1">9</span>)) WITH (DATA_SOURCE <span class="pl-k">=</span> CSE344_EXTERNAL);</pre>

<pre><span class="pl-k">SELECT</span> <span class="pl-c1">COUNT</span>(<span class="pl-k">*</span>) <span class="pl-k">FROM</span> Flights; <span class="pl-c">-- expect count of 1148675</span></pre>

<pre>$ mvn clean compile assembly:single</pre>

<pre>$ java -jar target/FlightApp-1.0-jar-with-dependencies.jar</pre>

<pre>$ mvn compile exec:java</pre>

<pre><span class="pl-c">// Generate a random cryptographic salt</span><span class="pl-smi">SecureRandom</span> random <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">SecureRandom</span>();<span class="pl-k">byte</span>[] salt <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">byte</span>[<span class="pl-c1">16</span>];random<span class="pl-k">.</span>nextBytes(salt);<span class="pl-c">// Specify the hash parameters</span><span class="pl-smi">KeySpec</span> spec <span class="pl-k">=</span> <span class="pl-k">new</span> <span class="pl-smi">PBEKeySpec</span>(password<span class="pl-k">.</span>toCharArray(), salt, <span class="pl-c1">HASH_STRENGTH</span>, <span class="pl-c1">KEY_LENGTH</span>);<span class="pl-c">// Generate the hash</span><span class="pl-smi">SecretKeyFactory</span> factory <span class="pl-k">=</span> <span class="pl-c1">null</span>;<span class="pl-k">byte</span>[] hash <span class="pl-k">=</span> <span class="pl-c1">null</span>;<span class="pl-k">try</span> {  factory <span class="pl-k">=</span> <span class="pl-smi">SecretKeyFactory</span><span class="pl-k">.</span>getInstance(<span class="pl-s"><span class="pl-pds">"</span>PBKDF2WithHmacSHA1<span class="pl-pds">"</span></span>);  hash <span class="pl-k">=</span> factory<span class="pl-k">.</span>generateSecret(spec)<span class="pl-k">.</span>getEncoded();} <span class="pl-k">catch</span> (<span class="pl-smi">NoSuchAlgorithmException</span> <span class="pl-k">|</span> <span class="pl-smi">InvalidKeySpecException</span> ex) {  <span class="pl-k">throw</span> <span class="pl-k">new</span> <span class="pl-smi">IllegalStateException</span>();}</pre>

<pre>$ mvn <span class="pl-c1">test</span></pre>

<pre>$ mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>folder_name_or_file_name_here<span class="pl-pds">"</span></span></pre>

<pre>[command 1][command 2]...<span class="pl-k">*</span>[expected output line 1][expected output line 2]...<span class="pl-k">*</span><span class="pl-c"># everything following ‘#’ is a comment on the same line</span></pre>

<pre>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/search<span class="pl-pds">"</span></span>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/login<span class="pl-pds">"</span></span>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/create<span class="pl-pds">"</span></span></pre>

<pre>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/search<span class="pl-pds">"</span></span>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/pay<span class="pl-pds">"</span></span>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/cancel<span class="pl-pds">"</span></span></pre>

<pre>mvn <span class="pl-c1">test</span> -Dtest.cases=<span class="pl-s"><span class="pl-pds">"</span>cases/no_transaction/<span class="pl-pds">"</span></span></pre>

<pre><span class="pl-c1">BEGIN</span> <span class="pl-c1">TRANSACTION</span><span class="pl-c1">....</span><span class="pl-c1">COMMIT</span> or <span class="pl-c1">ROLLBACK</span></pre>

<pre><span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-smi">String</span> <span class="pl-c1">BEGIN_TRANSACTION_SQL</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>BEGIN TRANSACTION;<span class="pl-pds">"</span></span>;<span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-smi">String</span> <span class="pl-c1">COMMIT_SQL</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>COMMIT TRANSACTION<span class="pl-pds">"</span></span>;<span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-smi">String</span> <span class="pl-c1">ROLLBACK_SQL</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>ROLLBACK TRANSACTION<span class="pl-pds">"</span></span>;<span class="pl-smi">Statement</span> generalStatement;<span class="pl-c">// When you start the database up</span><span class="pl-smi">Connection</span> conn <span class="pl-k">=</span> [<span class="pl-c1">...</span>]conn<span class="pl-k">.</span>setAutoCommit(<span class="pl-c1">true</span>); <span class="pl-c">// This is the default setting, actually</span>conn<span class="pl-k">.</span>setTransactionIsolation(<span class="pl-smi">Connection</span><span class="pl-c1"><span class="pl-k">.</span>TRANSACTION_SERIALIZABLE</span>);generalStatement <span class="pl-k">=</span> conn<span class="pl-k">.</span>createStatement();<span class="pl-c">// Transaction begin here</span>generalStatement<span class="pl-k">.</span>execute(<span class="pl-c1">BEGIN_TRANSACTION_SQL</span>);<span class="pl-c">// ... execute updates and queries.</span>generalStatement<span class="pl-k">.</span>execute(<span class="pl-c1">COMMIT_SQL</span>);<span class="pl-c">// OR</span>generalStatement<span class="pl-k">.</span>execute(<span class="pl-c1">ROLLBACK_SQL</span>);</pre>

<pre><span class="pl-c">// When you start the database up</span><span class="pl-smi">Connection</span> conn <span class="pl-k">=</span> [<span class="pl-c1">...</span>]conn<span class="pl-k">.</span>setAutoCommit(<span class="pl-c1">true</span>); <span class="pl-c">// This is the default setting, actually</span>conn<span class="pl-k">.</span>setTransactionIsolation(<span class="pl-smi">Connection</span><span class="pl-c1"><span class="pl-k">.</span>TRANSACTION_SERIALIZABLE</span>);<span class="pl-c">// In each operation that is to be a multi-statement SQL transaction:</span>conn<span class="pl-k">.</span>setAutoCommit(<span class="pl-c1">false</span>);<span class="pl-c">// You MUST do this in order to tell JDBC that you are starting a</span><span class="pl-c">// multi-statement transaction</span><span class="pl-c">// ... execute updates and queries.</span>conn<span class="pl-k">.</span>commit();<span class="pl-c">// OR</span>conn<span class="pl-k">.</span>rollback();conn<span class="pl-k">.</span>setAutoCommit(<span class="pl-c1">true</span>);<span class="pl-c">// You MUST do this to make sure that future statements execute as their own transactions.</span></pre>

<pre>[command 1][command 2]...<span class="pl-k">*</span>[expected output line 1][expected output line 2]...<span class="pl-k">*</span><span class="pl-c"># everything following ‘#’ is a comment on the same line</span></pre>