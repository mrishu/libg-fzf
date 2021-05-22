# genesisfzf
![](libg.gif)
![](libg_frame_35_delay-0.13s.gif)

**Library Genesis TUI** using **fzf**.  
Using this, you can search the library genesis site, filter your book out using `fzf` as well as download the selected book by just pressing `Return`; everything right from your terminal!

## Usage
`libg [OPTIONS] <search query>`

### Options:
 `-b <value>` 'Search By': `author`, `title`, `publisher`, `year`, `isbn`, `language`, `md5`, `tags`, `extension`.  
	`-n <value>`	'Number of Results': Any positive integer.  
	`-s <value>`	'Sort Results By': `id`, `author`, `title`, `publisher`, `year`, `pages`, `language`, `filesize`, `extension`.  
	`-o <value>`	'Sort Results in Order': `ASC`/`asc` (for ascending order), `DESC`/`desc` (for descending order).  
	
**NOTE**: `-n <value>` can take in any positive integer value, but can only produce results in numbers of 25, 50 or multiples of 100.  
      [1-25] will give 25 results, [26-50] will give 50 results, [50-100] will give 100 results, [101-200] will give 200 results, [201-300] will give 300 results and so on.  
      (Obviously upto available number of search results).

## Installation
Since it is just a small shell script, just download the script, give it executable permissions and place it in a directory that is in `PATH`. Also, you can change the default values for each option in the first few lines of the script.
 
## Dependency
The only dependency is `fzf`: https://github.com/junegunn/fzf.   
The other dependencies are `sed`(GNU), `awk`, `curl` and `wget` but are present by default in most Linux installs. 

## Todos
1. Add option for selecting different mirrors.
