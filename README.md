# CraigslistMinion
A minion to hunt Craigslist for you... First come, first serve means knowing puts you ahead of the curve!
#
# Sample Code
#!/usr/bin/ksh
#
# Quick little filter to hunt for items that match a given pattern on 
# good ole craigslist.  Keeps track of whats been seen, and emails along
# anything new to the provided address.  Scan frequency is set by the
# controller, aka, cron.
#
# by James 'Anjin' Hutchinson (c) 2008
#

#
# Search Pattern
QUERY="kayak"

#
# Email to send to
EMAIL="null@null.null"

#
# History File
HISTORY="kayak.urls"

#
# Craigslist URL
CRAIGSLIST="http://seattle.craigslist.org"

#
# Search postfix for URL
SEARCH="/search/sss?query="

#
# If first run, build the history file but send no email
RUN_ONCE="0"
if [ ! -f ${HISTORY} ]
then
	RUN_ONCE="1"
	echo "No history file!  Building history without sending alerts."
fi

#
# Primary filter 
GET ${CRAIGSLIST}${SEARCH}${QUERY} | gawk '
	BEGIN { 
		while( getline tstr < "'${HISTORY}'" > 0 ) {
			history[tstr] = tstr ;
			}
		RS="<p>" ;
		
		} 
	{ 
		month = $1 ;
		day = $2 ;
		split( $0, tary, "\<" ) ;
		split( tary[2], ttary, "\>" ) ;
		split( ttary[1], tttary, "\"" ) ;
		url = tttary[2] ;
		tcnt = split( ttary[2], ttttary, " - " ) ;
		desc = ttttary[1] ;
		for( i=2 ; i<tcnt ; i++ ) {
			desc = sprintf("%s %s",	desc, ttttary[i] ) ;
			}
		if( tcnt > 1 ) {
			price = ttttary[tcnt] ;
			}
		else {
			price = "No Price" ;
			}
		if( history[ url ] == "" ) {
			history[ url ] = url ;
			if( "'${RUN_ONCE}'" == "0" ) {
				syscmd = sprintf("echo \"%s%s \\%s '${CRAIGSLIST}'%s %s\" | \
					mailx -s \"'${QUERY}'\" '${EMAIL}'\n",
					month, day, price, url, desc )
				printf syscmd ;
				system(syscmd) ;
				close(syscmd) ;
				}
			printf "%s\n",url >> "'${HISTORY}'" ;
			}
		}' 2> /dev/null 
