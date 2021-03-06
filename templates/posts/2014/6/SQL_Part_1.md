{%block content%}
<h2>SQL What?</h2>
At some stage of a web application involving a SQL database, the query is broken down into a simple string. At this point, certain inputs can cause the SQL server (The reciving end) to confuse commands and values. This allows attackers to exfiltrate or manipulate data from your servers.


To demonstrate this, let's create a vulnerable web application.


<h2>Uber Secure Phonebook</h2>


    <h3 class="panel-heading">Schema:</h3>
    <pre class="prettyprint lang-sql">
    CREATE TABLE phonebook (
        id int AUTO_INCREMENT PRIMARY KEY NOT NULL,
        name varchar(255) NOT NULL,
        number int(11)
    );
    </pre>


<h3 class="panel-heading">Server:</h3>
    Okay, so I'm not going to write up the whole server here. If you want the code, I'll make a gist and link it over here. However, these are the important parts of the program.
    <div class="panel">
        <h4 class="panel-heading lang-sql">
            Adding an Entry:
        </h4>
        <pre class="prettyprint lang-sql">
        Conn.query("INSERT into phonebook (name, number) VALUES '"+name+"', '"+number)
        </pre>
        <h4 class="panel-heading lang-sql">
            Normal Lookup
        </h4>
        <pre class="prettyprint lang-sql">
        Conn.query("SELECT name, number FROM phonebook WHERE name LIKE '"+name+"'")
        </pre>
    </div>


<div class="panel well">
    <h3 class="panel-heading">
    The Fatal Flaw:
    </h3>
    When a non-malicious request is processed, the values are dynamically added into the query using simple string concatenation. For example, if we were to look up Joe Bob's phone number, the query used would be:
    <pre class="prettyprint lang-sql">
    SELECT name, number FROM phonebook WHERE name LIKE 'Joe Bob';
    </pre>
    <p>
    However, because this query is a simple string at this point, odd inputs can make our program get confused.
    </p>
    <p>
    Take the case of our Irish friend, Mr. O'Malley. In his case, the query would become:
    </p>
    <pre class="prettyprint lang-sql">
    SELECT name, number FROM phonebook WHERE name LIKE 'Mr. O'Malley';
    </pre>
    Wait a sec. That doesnt look right. Why's that coloring off?
    <h3>
        Welcome to SQL Injection:
    </h3>
    <p>
    Because this query is being transmitted to our database as a string, our database will have just as much trouble realizing that the "Malley" in Mr. O'Malley's name is not actually part of the query as my syntax highlighter. In fact, if we were to run this query, we would get a syntax error, even though our query syntax is fine, our data is just wrong.
    </p>
    <p>
    So what hilariousness can we cause here? Well, now we have control over the SQL statement. We can redirect it however we want to. For example, this really irritating phone salesman has been bothering me, and I have no idea who he is. I sure would like to find out, though.Using this vulnerability, we actually can. If we input the name we want to find as:
    <pre class="prettyprint lang-sql">
    doesnt_matter' OR number=12345678910 #
    </pre>
    Our query now becomes:
    <pre class="prettyprint lang-sql">
    SELECT name, number FROM phonebook WHERE name LIKE 'doesnt_matter' OR number=12345678910 #'
    </pre>
    To make this a little clearer, let's add parentheses.
    <pre class="prettyprint lang-sql">
    SELECT name, number FROM phonebook WHERE (name LIKE 'doesnt_matter') OR (number=12345678910) #'
    </pre>
    </p>
    Okay. Now things are a little clearer. Our injected data split the query in the WHERE statement up into 2 halves. Assuming that there is nobody in the database with the name "doesnt_matter", the only people returned from this query will have the phone number that we're looking for. Now the next time our telemarketer calls, we'll have a nice chance to surprise him.
</div>

{%end%}
