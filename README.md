NAME
====

`pg_hunspell` - adding a language to PostgreSQL's (Full Text Search) FTS with Hunspell

SYNOPSIS
===

    pg_hunspell_install pl PL polish

DESCRIPTION
====

This utility does all the leg work. [To add a language dictionary to your PostgreSQL's instance](https://www.postgresql.org/docs/current/static/textsearch-dictionaries.html), you'll need three dictionary files:

* `.dict`
* `.affix`
* `.stop`

We download them and produce the SQL to help get them functioning as a dictionary, and FTS configuration in the database.

Obtaining dictionary files
--------------------------

We have two sources,
	 

* On Debian or Ubuntu the script will use `apt` to install the files, or `dpkg` if they're already installed.
* Or, if you're not on Debian or Ubuntu, it will source them from the proper projects on github. The dictionary and affix files we get from [LibreOffice Dictionaries](https://github.com/LibreOffice/dictionaries), and we pull the stopwords from the [stopwords-iso](https://github.com/stopwords-iso) project.

Once this is done, those files get processed and installed into the `tsearch_data` dir belonging to your PostgreSQL's installation. You can find the location of that directory with `pg_config --sharedir`. If you're on Debian or Ubuntu, you'll be prompted to do this automatically. If you wish to install for other version of PostgreSQL, you'll have to copy the above listed files to the appropriate `tsearch_data` location.

Creating search dict and config in PostgreSQL
---------------------------------------------

After you run the `pg_hunspell_install` script the SQL to `CREATE` the DICTIONARY and CONFIGURATION will be outputted, as well as the catalog annotations. Simply start `psql` and run these commands.

Testing
-------

Postgres comes with `ts_debug` function that's useful for testing text search configs. Lets test some random phrase:

    SELECT token, dictionary, lexemes
    FROM ts_debug(
      'polish',
      'Szybkie brązowe lisy przeskoczyły ponad starym szarym Burkiem który spal.'
    )
    WHERE alias <> 'blank';

Here's the output:

        token     | dictionary  |      lexemes        
    --------------+-------------+---------------------
     Szybkie      | polish_dict | {szybki}
     brązowe      | polish_dict | {brązowy}
     lisy         | polish_dict | {lisa,lis}
     przeskoczyły | polish_dict | {przeskoczyć}
     ponad        | polish_dict | {}
     starym       | polish_dict | {starym,stary,stara}
     szarym       | polish_dict | {szary}
     Burkiem      | polish_dict | {burkiem,burek}
     który        | polish_dict | {}
     spał         | polish_dict | {spała,spać}

This shows that the dict we've just added, the `polish_dict`, was used and valid lexemes were resolved for each of words used, meaning that search for `szybki lis` would've matched `szybkie lisy`.

**GOTCHA**: You may need to [additionally unaccent your strings](https://www.postgresql.org/docs/current/static/unaccent.html) if you want to search without diacritics, otherwhise your search my return surprising results.

If you want for searches for, say, `iphone` to match `apple phone`, you should research the thesaurus features.

SEE ALSO
=====

Something I didn't know when I endeavoured to start this, Debian provides
`pg_updatedicts`. Currently, it does some things we don't with the update
mechanisms. That is to say, in some ways it is better. Given enough time, I'll
steal those ideas. They don't do stop words, and I'm going to get a thesaurus
up too. After which, I'll submit to mainline PostgreSQL; and, also to Debian to
see if they're interested in upgrading.

SOURCE CODE
=====

Find the code on the [GitHub Repository](https://github.com/EvanCarroll/pg_hunspell). PR's welcome!

AUTHORS
====

 * Evan Carroll, 14 Nov 2017
 * Rafał Pitoń, 09 Aug 2016
