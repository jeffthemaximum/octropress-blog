---
layout: post
title: "Intro to SQLite3 with Python's Flask"
date: 2015-07-24 10:59:45 -0400
comments: true
categories: technical
---

I've got a URL shortening website like [Bitly](http://www.bitly.com) that allows you to submit a link, then gives you a new, hopefully shorter url to use for that link. I built it with Python and the [Flask microframework](http://flask.pocoo.org/).
<!-- more -->
You can checkout the live site here: [tinyjeff.herokuapp.com](http://tinyjeff.herokuapp.com).

Right now, a user enters a URL at my flask app, such as "www.jeffreymaxim.com". My python code will generate a random key for that site, such as "98rDv64O". It will store both the original URL and the key in a python dictionary like so:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>my_links[&quot;www.jeffreymaxim.com&quot;] = &quot;98rDv64O&quot;
</code></pre>

So that I've got an entry in the my_links dictionary that looks like this:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>{'www.jeffreymaxim.com': '98rDv64O'}
</code></pre>

This code allows me to redirect a user from [http://tinyjeff.herokuapp.com/98rDv64O](http://tinyjeff.herokuapp.com/98rDv64O) to [www.jeffreymaxim.com](http://www.jeffreymaxim.com).

The problem here is that this dictionary only lives in the local memory of my machine while my app is running. As soon as I restart my server, that whole dictionary and all my links and keys gets deleted. 

The solution is to implement a database.
<h2 margin='0' padding='0'>SQLite3</h2>

Enter SQLite3. This is essentially an excel spreadsheet for your website. It allows you to store tables, columns, and rows of data that is saved to disk. My previous dictionary would get deleted in my server restarted. SQLite3 will allow my to save my database even if my server restarts. This was, I can save my URL's and keys indefinitely.

To setup SQLite3 for my Bitly-clone website, I need to do three things:

1. A schema.sql file that defines the structure of my database.
2. A models.py file that defines my functions to inset, query, and update entries in my database.
3. Updates to my main python code so that storage in the my_links dictionary is replaced with storage in the SQLite3 database.

Let's go through each...
<h2 margin='0' padding='0'>A schema.sql file that defines the structure of my database.</h2>

The first step is to define the structure of my database using a schema.sql file that I create in the root directory of my flask app. The contents on the schema.sql file looks like this:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>drop table if exists link;
    create table link (
        id integer primary key autoincrement,
        key text not null,
        url text not null,
        hits integer not null
);
</code></pre>

This SQL code says to create a database called "link" with four columns, id, key, url, and hits. If you think of the SQLite3 database like an excel spreadsheet, then we have a spreadsheet table with the four columns, and each row will store entries into the database.

Next, we need to use this schema.sql file to create our database by running the following command from the terminal while in the root directory of the app:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>sqlite3 database.db &lt; schema.sql
</code></pre>

This tells SQLite3 to create a file called database.db which is built using the structure we outlined in schema.sql.
<h2 margin='0' padding='0'>A models.py file that defines my functions to inset, query, and update entries in my database.</h2>

Now that we've created our empty database in the database.db file, I need to create python functions to interact with that database. I'm going to create a query, insert, and update function that will allow my to do those three respective things. My three functions look like this:

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>import sqlite3 as sql

def insert_link(key,url,hits):
    with sql.connect(&quot;database.db&quot;) as con:
        cur = con.cursor()
        cur.execute(&quot;INSERT INTO link (key,url,hits) VALUES (?,?,?)&quot;, (key,url,hits))
        con.commit()

def query_hits_and_url_for_link(key):
    with sql.connect(&quot;database.db&quot;) as con:
        cur = con.cursor()
        key = str(key)
        cur.execute(&quot;SELECT * FROM link WHERE key=?&quot;, (key,))
        link_data = cur.fetchall()
        print link_data
        return link_data

def update_hits(key):
    with sql.connect(&quot;database.db&quot;) as con:
        cur = con.cursor()
        cur.execute(&quot;UPDATE link SET hits = (hits + 1) WHERE key=key&quot;)
        con.commit()
</code></pre>

If we just look at the insert_link function and go through it line-by-line...

<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>with sql.connect(&quot;database.db&quot;) as con:
</code></pre>
This connects to the SQL database.
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>cur = con.cursor()
</code></pre>
Creates a cursor object called "cur".
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>cur.execute(&quot;INSERT INTO link (key,url,hits) VALUES (?,?,?)&quot;, (key,url,hits))
</code></pre>
This line executes our SQL statement and sends it to the database.
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>con.close()
</code></pre>
Finally, we commit our changes to the database.

The other two functions follow similar logic.
<h2 margin='0' padding='0'>Updates to my main python code so that storage in the my_links dictionary is replaced with storage in the SQLite3 database.</h2>

Before I had my SQLite3 database, my main python code would insert URL's and key's into a dictionary. Now we need to replace that logic with python code that enters the URL's and keys into the database.

When a user enters a URL into [tinyjeff.herokuapp.com](http://tinyjeff.herokuapp.com), I know enter that URL and key into my database by executing the insert_url function described above:
<pre style="font-family: Andale Mono, Lucida Console, Monaco, fixed, monospace; color: #000000; background-color: #eee;font-size: 12px;border: 1px dashed #999999;line-height: 14px;padding: 5px; overflow: auto; width: 100%"><code>models.insert_link(key, url, 0)
</code></pre>
Now, when a user enters [www.jeffreymaxim.com](http://www.jeffreymaxim.com) to my website, and my python code generates a key of "98rDv64O", this insert_key function will insert the URL and key into my database as rows. The 0 parameter in the insert_key function tells my database that this URL initially has zero hits.

The query function described above and the update_hits function described above can similarly be used to lookup information from my database and to update information in my database when this is required.
<h2 margin='0' padding='0'>Conclusion</h2>

SQLite3 allows you to create a more permanent database for your website to store data even if your server resets, or even if a user refreshes a webpage. You can use it to store data in an excel like fashion. Follow my directions above to try to implement a SQLite3 database in your python and flask website!