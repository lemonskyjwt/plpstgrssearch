Adding Polish language to PostgreSQL's Search
=============================================

Obtaining dictionary files
--------------------------

To add Polish language to your PostgreSQL's instance, you'll need three dictionary files:

* polish.dict
* polish.affix
* polish.stop

The last one (``polish.stop`` containing stopwords) is already supplied, but the other two need to be generated.

To get them you'll need to run the ``getdicts`` utility script writen in bash that downloads hunspell dictionary files from the ``src.chromium.org`` and mangles them into dict and affix files suitable for PostgreSQL.

Once this is done, those files should be automatically installed into the ``tsearch_data`` dir belonging to your PostgreSQL's installation. You can find the location of that directory with ``pg_config --sharedir``. If you wish to install for other version of PostgreSQL, you'll have to copy the above listed files to the appropriate ``tsearch_data`` location.


Creating search dict and config in PostgreSQL
---------------------------------------------

Now start the ``psql`` utility. You'll need to run three SQLs:

* create search dictionary for the language
* copy default config for English language to Polish
* associate newly created dict and config with each other

First one will use the files we've generated in previous step to define new dictionary in system:

    CREATE TEXT SEARCH DICTIONARY polish_dict (
        TEMPLATE = ispell,
        DictFile = polish,
        AffFile = polish,
        StopWords = polish
    );

Second one is pretty straightfoward:

    CREATE TEXT SEARCH CONFIGURATION public.polish ( COPY = pg_catalog.english );

This will create ``public.polish`` configuration. Now associate one with another:

    ALTER TEXT SEARCH CONFIGURATION polish
        ALTER MAPPING
            FOR
                asciiword, asciihword, hword_asciipart,  word, hword, hword_part
            WITH
                polish_dict, simple;

That's it! If no error was brought up, you are ready to test if your new config works.


Testing
-------

Postgres comes with ``ts_debug`` function that's useful for testing text search configs. Lets test some random phrase:

    SELECT token, dictionary, lexemes FROM ts_debug(
        'public.polish',
        'Szybkie brązowe lisy przeskoczyły ponad starym szarym Burkiem który spal.'
    ) where alias <> 'blank';

Here's the output:

        token     | dictionary  |      lexemes        
    --------------+-------------+---------------------
     Szybkie      | polish_dict | {szybki}
     brązowe      | polish_dict | {brązowy}
     lisy         | polish_dict | {lisa,lis}
     przeskoczyły | polish_dict | {przeskoczyć}
     ponad        | polish_dict | {ponad}
     starym       | polish_dict | {starym,stary,stara}
     szarym       | polish_dict | {szary}
     Burkiem      | polish_dict | {burkiem,burek}
     który        | polish_dict | {który}
     spał         | polish_dict | {spała,spać}

This shows that the dict we've just added, the ``polish_dict``, was used and valid lexemes were resolved for each of words used, meaning that search for ``szybki lis`` would've matched ``szybkie lisy``.

**GOTCHA**: You may need to additionally unaccent your strings if you expect your users to search without diacritics, otherwhise your search my return suprising results, like ``spal`` being interpreted for ``spalić`` instead of ``spać``. Likewise you'll need to develop (likely domain specific) dict of synonyms if you want for searches for, say, ``iphone`` to match ``apple phone``.


Endnotes
========

Following guide was tested on PostgreSQL 9.5. Eventually I would like to include synonym and unaccent dicts howto's too and maybe move utility script to Python instead. PR's accepted.

Rafał Pitoń, 09 Aug 2016
