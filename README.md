![](libg.gif)
# genesisfzf
**Library Genesis TUI** using **fzf**.  
Using this, you can search the library genesis site, filter your book out using `fzf` as well as download the selected book by just pressing `Return`; everything right from your terminal!

## Usage
`libg search terms`  
For e.g.  
`libg curves and surfaces`.   
Or just entering `libg` brings up a `Enter searchterm:` prompt where you can enter search terms.
 
To change the number of pages to be scraped, change the `DEPTH` variable to whatever you want.  
Similarly, change `SORT`, `SORTMODE` and `SEARCHBY` variables to change other settings. These with all its possible values are listed in the top few lines.  

## Installation
Since it is just a small shell script, just download the script, give it executable permissions and place it in a directory that is in `PATH`.
 
## Dependency
The only dependency is `fzf`: https://github.com/junegunn/fzf.   
The other dependencies are `sed`(GNU), `awk`, `curl` and `wget` but are present by default in most Linux installs. 

## Todos
1. Add command line options for changing `SORT`, `SORTMODE`, `DEPTH`, `SEARCHBY`.
2. Add caching option. For e.g. if someone has searched something and scraped some pages already, then data of those pages can be cached for future use. If suppose the user searches for the same search term again with the same options, then a prompt may come up asking if the user wants to use cached data. If the `DEPTH` of the current search is less than the `DEPTH` of cached search, then the cached search will be used. If the `DEPTH` of current search is more, the scraping will start from the end of the cached search.
