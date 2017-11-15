Adding a language to PostgreSQL's FTS with Hunspell
=============================================

Obtaining dictionary files
--------------------------

To add a language to your PostgreSQL's instance, you'll need three dictionary files:

* .dict
* .affix
* .stop

For polish, the last one (`polish.stop` containing stopwords) is already supplied, but the other two need to be generated.  To get them you'll need to run the `getdicts` shell script. This will obtain the appropriate files as produced by [LibreOffice Dictionaries](https://github.com/LibreOffice/dictionaries)

* On Debian or Ubuntu the script will use `apt` to install the files, or `dpkg` if they're already installed.
* Or, if you're not on Debian or Ubuntu, it will source them from the LibreOffice source repo.

Once this is done, those files should be installed into the ``tsearch_data`` dir belonging to your PostgreSQL's installation. You can find the location of that directory with ``pg_config --sharedir``. If you're on Debian or Ubuntu, you'll be prompted to do this automatially. If you wish to install for other version of PostgreSQL, you'll have to copy the above listed files to the appropriate ``tsearch_data`` location.


Creating search dict and config in PostgreSQL
---------------------------------------------

After you run the `./getdicts` script the SQL to `CREATE` the DICTIONARY and CONFIGURATION, annoted should be outputted. Simply start psql and run these commands.



Testing
-------

Postgres comes with `ts_debug` function that's useful for testing text search configs. Lets test some random phrase:

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

Please send PR's on Github.

Evan Carroll, 14 Nov 2017
Rafał Pitoń, 09 Aug 2016
