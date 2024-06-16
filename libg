#!/bin/sh

CONFIG_DIR="$HOME/.config/libg"
CONFIG_FILE="$CONFIG_DIR/libg.sh"
if [ -f "$CONFIG_FILE" ] ; then
	. "$CONFIG_FILE"
else 
	mkdir -p "$CONFIG_DIR" > /dev/null
	printf "DEFAULT_RESPERPAGE=100 # default number of search results per page (allowed values: 25, 50, 100)\
	\nDEFAULT_DEPTH=1 # default number of result pages to scan (allowed values: any positive integer)\
	\nLIBGEN_MIRROR=\"https://libgen.rs\" # alternatives: *.is, *.rs, *.st\
	\nDOWNLOAD_LOCATION=\"\" # default download location (if empty then books will be downloaded to current directory)" > "$CONFIG_FILE"
	. "$CONFIG_FILE"
fi

[ -z "$DOWNLOAD_LOCATION" ] || [ -d "$DOWNLOAD_LOCATION" ] && cd "$DOWNLOAD_LOCATION"

basic_help() {
	printf "libg: Shell Script to scrape library genesis website and download books right from the terminal.\
	\nUSAGE: libg [OPTIONS] <search query>\
	\n\nOPTIONS        Description                               Allowed Values\
	\n-------        -----------                               --------------\
	\n-b <value>     Search By                                 author, title, publisher, year, isbn, language, md5, tags, extension.\
	\n-n <value>     Number of Search Results per Page         25, 50, 100.\
	\n-d <value>     Depth (Number of result pages to scan)    Any positive integer.\
	\n-s <value>     Sort Results By                           id, author, title, publisher, year, pages, language, filesize, extension.\
	\n-r             Reverse/Desecending Order\
	\n\nCurrent Defaults (Can be changed by editing the script)\
	\n----------------\
	\nNumber of Search Results per Page       $DEFAULT_RESPERPAGE\
	\nDepth                                   $DEFAULT_DEPTH\
	\nLibgen Mirror                           $LIBGEN_MIRROR\n"
}

# -- PARSE OPTIONS --
while getopts b:n:d:s:rh opt; do
	case "$opt" in
		# SEARCHBY: author, title, publisher, year, isbn, language, md5, tags, extension
		b) SEARCHBY="$OPTARG" ;;
		# RESPERPAGE: 25, 50, 100
		n) RESPERPAGE="$OPTARG" ;;
		# DEPTH: Any positive integer
		d) DEPTH="$OPTARG" ;;
		# SORTBY: id, author, title, publisher, year, pages, language, filesize, extension
		s) SORTBY="$OPTARG" ;;
		# SORTORDER: ASC, DESC
		r) SORTORDER=DESC ;;
		h) basic_help && exit 0 ;;
		\?) basic_help && exit 1 ;;
	esac
done
shift $(( OPTIND-1 ))

# -- CHECK VALIDITY OF OPTIONS --
case "$SEARCHBY" in
	"") ;; author) ;; title) ;; publisher) ;; year) ;; isbn) ;; language) ;; md5) ;; tags) ;; extension) ;;
	*) printf "\033[31mERR\033[0m: Invalid Value for -b option!\nAllowed values: author, title, publisher, year, isbn, language, md5, tags, extension.\n" && exit 1 ;;
esac

case "$RESPERPAGE" in
	"") RESPERPAGE="$DEFAULT_RESPERPAGE" ;;
	0) printf "\033[31mERR\033[0m: Invalid Value for -n option!\nAllowed values: 25, 50, 100.\n" && exit 1 ;;
	[0-9]*) ;;
	*) printf "\033[31mERR\033[0m: Invalid Value for -n option!\nAllowed values: 25, 50, 100.\n" && exit 1 ;;
esac

case "$DEPTH" in
	"") DEPTH="$DEFAULT_DEPTH" ;;
	0) printf "\033[31mERR\033[0m: Invalid Value for -d option!\nAllowed values: Any positive integer.\n" && exit 1 ;;
	[0-9]*) ;;
	*) printf "\033[31mERR\033[0m: Invalid Value for -d option!\nAllowed values: Any positive integer.\n" && exit 1 ;;
esac

case "$SORTBY" in
	"") ;; id) ;; author) ;; title) ;; publisher) ;; year) ;; pages) ;; language) ;; filesize) ;; extension) ;;
	*) printf "\033[31mERR\033[0m: Invalid Value for -s option!\nAllowed values: id, author, title, publisher, year, pages, language, filesize, extension.\n" && exit 1 ;;
esac

case "$SORTORDER" in
	"") ;; ASC) ;; DESC) ;;
	*) printf "\033[31mERR\033[0m: Invalid value given for SORTORDER." && exit 1 ;;
esac


# -- GET SEARCH TERMS --
QUERY="$*"
if [ -z "$QUERY" ]; then
	printf "Enter query: "
	read -r QUERY
fi

# -- SCRAPING --
page=1
original_page="$(mktemp)"
data_file="$(mktemp)"
while [ "$page" -le "$DEPTH" ]; do
	printf "\033[1mGetting Page \033[35m$page... \033[0m"

	curl -s -G "$LIBGEN_MIRROR/search.php" \
		--data-urlencode "req=$QUERY" \
		--data-urlencode "column=$SEARCHBY" \
		--data-urlencode "res=$RESPERPAGE" \
		--data-urlencode "page=$page" \
		--data-urlencode "sort=$SORTBY" \
		--data-urlencode "sortmode=$SORTORDER" > "$original_page"
	status="$?"
	[ "$status" -eq 6 ] && printf "\033[31mFailed!\nERR\033[0m: Couldn't connect to Library Genesis! Check you internet connection and try again!\n" && exit 2

	printf "\033[32mCompleted!\033[0m\n"
	printf "\033[1mScraping Page \033[35m$page... \033[0m"
	if grep -q '<td><b>Edit</b></td></tr></tr></table>' "$original_page"; then
		printf "\033[1mEnd of Results! Nothing on page \033[35m$page!\033[0m\n"
		[ "$page" -eq 1 ] && exit
		break
	fi
	# Extract the relevant part. Note that we are making only one pass through the file :)
	sed -i -n '/^<td><b>Edit.*/,$p;/^<\/tr><\/table>$/q' "$original_page"
	# -- Cleaning up --
	# Remove unnecessary part from first line and delete last line
	sed -i '1s_.*<td_<td_;$d' "$original_page"
	# Delete lines of closing </tr> tags
	sed '/^\t*<\/tr>$\|^$/d' "$original_page" |\
	# Remove opening <td> and closing </td> tags along with \r's
	sed 's_\t*</\?td[^>]*>\r\?__g' |\
	# -- Extracting authors --
	sed 's_<a [^>]*author["'\'']>\([^<]*\)</a>_\1_g' |\
	# -- Extracting bookname --
	# Remove <font> and <i> tags first.
	# Now a author line can consist of three things: series, bookname, ISBNs separated by commas; and all of them separated by <br> tags.
	# Bookname is a must but others may or may not be there. So, convert the <br> tags to \n so that we can work on them separately.
	sed 's_</\?font[^>]*>__g' | sed 's_</\?i>__g' | sed '/<a href=['\''"]book\/index\.php/s_<br>_\n_g' |\
	# Prepend the series lines with \x01 to mark them and move them to last at the end. And remove the <a> tags around them.
	sed '/<a href=[^>]*&column=series/s_^_\x01_' | sed '/<a href=[^>]*&column=series/s_</\?a[^>]*>__g' |\
	# Prepend the book lines with \x02 to mark them. And remove the <a> tags around them.
	sed '/<a href=['\''"]book\/index\.php/s_^_\x02_' | sed '/<a href=['\''"]book\/index\.php/s_</\?a[^>]*>__g' |\
	# Prepend the ISBNs with \x03 to mark them and move them to last at the end. Also remove the </a> tags which are at the end.
	# Its the only one that starts with a space.
	sed '/^\s/s_^\s\(.*\)</a>_\x03\1_' |\
	# -- Extracting links --
	# Now remaining <a href...> tags are one with the links. Extract links and remove [edit] link which is meant for Libgen Librarian.
	sed 's_<a[^>]*href='\''\([^'\'']*\)'\''[^>]*[^<]*</a>_\1 _g' | sed '/https:\/\/library.bz\/main\/edit.*/d' |\
	# -- Cleanup --
	# Convert &amp to ',', convert <br> tags and &nbsp to spaces.
	sed 's_&amp;_,_g;s_<br>_ _g;s_&nbsp;_ _g' |\
	# Convert all newlines to tabspaces and all opening <tr> tags to newlines to separate books. Also remove trailing whitespaces.
	tr '\n' '\t' | sed 's_<tr[^>]*>_\n_g' | sed 's_\s*$__' |\
	# -- Moving Series and ISBNs to last --
	sed 's_\(\t\x01[^\t]*\)\?\(\t\x02[^\t]*\)\(\t\x03[^\t]*\)\?\(.*\)_\2\4\t\1\3_' |\
	sed 's_\t\x01_\x01_' >> "$data_file"
	printf "\n" >> "$data_file"
	printf "\033[32mCompleted!\033[0m\n"
	page=$((page+1))
done

# -- DISPLAYING IN FZF --
# ID, Author, Title, Publisher, Year, Pages, Language, Size, Extension, Links, [Series], [ISBNs] -> 1,2,3,4,5,6,7,8,9,10,11,12
id=$(awk -F "\t" '{printf("%-30.30s | %-70.70s | %-16.16s | %-7.7s | %-4.4s | %-s\n", $2, $3, $5, $8, $9, $1)}' "$data_file" |\
	fzf  --reverse --ansi --preview 'awk -F"\t" '\''$1 == '\''{-1}'\'' \
	{printf("\033[1m\033[31mID:\033[0m %s\
		\n\033[1m\033[31mAuthors:\033[0m %s\
		\n\033[1m\033[31mTitle:\033[0m %s\
		\n\t\033[1m\033[35mSeries:\033[0m %s\
		\n\t\033[1m\033[35mISBNs:\033[0m %s\
		\n\033[1m\033[31mPublisher:\033[0m %s\
		\n\033[1m\033[31mYear:\033[0m %s\
		\n\033[1m\033[31mPages:\033[0m %s\
		\n\033[1m\033[31mLanguage:\033[0m %s\
		\n\033[1m\033[31mSize:\033[0m %s\
		\n\033[1m\033[31mExtension:\033[0m %s", $1,$2,$3,$11,$12,$4,$5,$6,$7,$8,$9)}'\'' '"$data_file"''\
	--preview-window=up:35% |\
	awk '{print $NF}')
[ -z "$id" ] && rm -f "$original_page" "$data_file" && exit 0

# -- DOWNLOADING --
download_page="$(mktemp)"
dl_links=$(awk -F"\t" '$1=="'"$id"'" {print $10}' "$data_file")
final_link=$(printf "$dl_links" | tr ' ' '\n' | awk '{printf "[%d]\t%s\n", NR, $0}' |  fzf --reverse | awk '{print $NF}')
[ -z "$final_link" ] && rm -f "$original_page" "$data_file" "$download_page" && exit 0
curl -s -L "$final_link" > "$download_page"

# Get the main download link [GET]
GET_URL=$(sed -n '/GET/s_.*<a href=["'\'']\(.*\)["'\'']>.*GET.*</a>.*_\1_p' "$download_page")

# Give option to select ipfs gateways (Cloudflare, IPFS.io, Crust, Pinata) if first link selected
if printf "%s" "$final_link" | grep -q 'library.lol'; then
	ipfs_gateways=$(sed -n '/Cloudflare/p' "$download_page" | sed 's_</li>_\n_g' | sed 's_.*<a href="\(.*\)">\(.*\)</a>.*_\2\t\1_;$d')
	DOWNLOAD_URL=$(printf "[GET]\t%s\n%s" "$GET_URL" "$ipfs_gateways" |\
		awk -F"\t" '{printf("%-20.20s %s\n", $1, $2)}' | fzf --reverse | awk '{print $NF}')
	[ -z "$DOWNLOAD_URL" ] && rm -f "$original_page" "$data_file" "$download_page" && exit 0
	wget --content-disposition "$DOWNLOAD_URL"
else
	wget --content-disposition "https://libgen.rocks/$GET_URL"
fi

rm -f "$original_page" "$data_file" "$download_page"

printf "\nHere is the final link in case it didn't work:\n%s\n" "$final_link"
