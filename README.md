# library_genesis-fzf (libg-fzf)
![](libg.gif)

**Library Genesis TUI** using **fzf**.  
Using this, you can search the library genesis site (https://libgen.is), filter your book out using `fzf` as well as download the selected book by just pressing `Enter`; everything right from your terminal!

## Usage
`libg [OPTIONS] <search query>`

### Options:
```
OPTIONS        Description                               Allowed Values
-------        -----------                               --------------
-b <value>     Search By                                 author, title, publisher, year, isbn, language, md5, tags, extension.
-n <value>     Number of Search Results per Page         25, 50, 100.
-d <value>     Depth (Number of result pages to scan)    Any positive integer.
-s <value>     Sort Results By                           id, author, title, publisher, year, pages, language, filesize, extension.
-r             Reverse/Desecending Order
```

## Config file
Edit `$HOME/.config/libg/libg.sh` to change default values:

```bash
DEFAULT_RESPERPAGE=100 # default number of search results per page (allowed values: 25, 50, 100)
DEFAULT_DEPTH=1 # default number of result pages to scan (allowed values: any positive integer)
LIBGEN_MIRROR="https://libgen.rs" # alternatives: *.is, *.rs, *.st	
DOWNLOAD_LOCATION="" # default download location (if empty then books will be downloaded to current directory)
```

**Note**: Giving any value except for 25, 50 or 100 for `DEFAULT_RESPERPAGE` will default it to 25.

## Installation
Since it is just a small shell script, just download the script, give it executable permissions and place it in a directory that is in `PATH`.
## Dependency
The only dependency is `fzf`: https://github.com/junegunn/fzf.   
The other dependencies are `sed`(GNU), `awk`, `curl` and `wget` but are present by default in most Linux installs. 
