#!/bin/sh

if [ -z "$1" -o -z "$2" -o -z "$3" ]; then
  cat <<EOF
getdicts lang2 country language

	lang2    - the two letter abbreviation of the language ISO 3166-1 ALPHA-2
	country  - the two letter abbreviation of the country ISO 639
	language - the full name of the language

	ex, getdicts en US english
EOF
	exit 1;
fi

# ISO639
LANG="$1"

# ISO 3166-1 ALPHA-2
LOCALE="$2"

# Full Language Name
LANGFULL="$3"

ENCODING="ISO8859-2"
IDENTIFIER="${LANG}_${LOCALE}"


echo "Creating PostgreSQL dictionary files for \"$IDENTIFIER\"";
mkdir -p ./dist ./temp;

## Use local files if we can and if they exist.
if command -v apt-get > /dev/null; then
	echo "Debian/Ubuntu detected - using apt";
	PACKAGE="hunspell-$LANG"

  if ! dpkg -V "$PACKAGE"; then
		echo "Installing through apt-get"
		sudo apt-get install --yes "$PACKAGE";
	fi;

	for ext in aff dic; do
		echo "Using local $IDENTIFIER.$ext";
		SYSTEM_FILE=$(dpkg -L "$PACKAGE" | grep "$IDENTIFIER.$ext");
		iconv -f "$ENCODING" -t utf8 -o "./temp/$IDENTIFIER.utf8.$ext" "$SYSTEM_FILE";
	done

## Or we can download them..
else
	for ext in aff dic; do
		if [ ! -e "./temp/$IDENTIFIER.utf8.$ext" ]; then
			## Download and convert to UTF-8 from $ENCODING
			wget --no-verbose "https://raw.githubusercontent.com/LibreOffice/dictionaries/master/$IDENTIFIER/$IDENTIFIER.$ext" -O - |
			iconv -f "$ENCODING" -t utf8 -o "./temp/$IDENTIFIER.utf8.$ext";
		fi;
	done;
fi

## Download the stop words if they don't exist, no one packages them
if [ ! -e "./dist/$LANGFULL.stop" ]; then
	wget --no-verbose "https://raw.githubusercontent.com/stopwords-iso/stopwords-$LANG/master/stopwords-$LANG.txt" -O "./dist/$LANGFULL.stop";
fi

## also a bit of processing.. 
# 1. PostgreSQL wants .affix and .dict,
#    LibreOffice uses .aff and .dic
# 2. Remove first line of dic (from docs)
#    > The first  line of the dictionaries (except personal dictionaries) contains
#    > the approximate word count (for optimal hash memory  size).
# 3. Removes blank lines (if they exist) and sorts dict
sed -e '1d' -e '/^$/d' "./temp/$IDENTIFIER.utf8.dic" | sort > "./dist/$IDENTIFIER.dict"
cp "./temp/$IDENTIFIER.utf8.aff" "./dist/$IDENTIFIER.affix"

# Offer to install in Debian/Ubuntu
if command -v apt-get > /dev/null 2>&1 &&
 	command -v pg_config > /dev/null 2>&1;
then

	PG_TSEARCH_DATA="$(pg_config --sharedir)/tsearch_data";
	PG_VERSION=$(pg_config --version);
	echo "Attempting install for $PG_VERSION to $PG_TSEARCH_DATA";
	sudo cp -v                   \
		"./dist/$LANGFULL.stop"    \
		"./dist/$IDENTIFIER.affix" \
		"./dist/$IDENTIFIER.dict"  \
		"$PG_TSEARCH_DATA";
	cat <<EOF


--
-- Run this on the database
--
CREATE TEXT SEARCH DICTIONARY ${LANGFULL}_hunspell (
	TEMPLATE  = ispell,
	DictFile  = $IDENTIFIER,
	AffFile   = $IDENTIFIER,
	StopWords = $LANGFULL
);
COMMENT ON TEXT SEARCH DICTIONARY ${LANGFULL}_hunspell
	IS '[USER ADDED] Hunspell dictionary for $LANGFULL';
CREATE TEXT SEARCH CONFIGURATION public.$LANGFULL (
	COPY = pg_catalog.english
);
ALTER TEXT SEARCH CONFIGURATION $LANGFULL
	ALTER MAPPING
	FOR
		asciiword, asciihword, hword_asciipart,  word, hword, hword_part
	WITH
		${LANGFULL}_hunspell, simple;
COMMENT ON TEXT SEARCH CONFIGURATION $LANGFULL
	IS '[USER ADDED] configuration for $LANGFULL';
EOF

fi;

echo "Finished!"
