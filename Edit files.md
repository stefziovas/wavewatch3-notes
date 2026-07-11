## How to quick search in vi

- ```Esc + gg``` : Go to top the file.
- ```Esc + G``` : Go to bottom of the file.
- ```Esc + Ctrl + End``` : Jump end of file.
-  ```Esc + Ctrl + Home``` : Jump start of file.
- Search term: ```/term_you_want_to_search```
Then navigate through file via this text using ```n``` to go to next place this term is found, and ```N``` to go to the previous place it was found
- Search content no matter if it's in upper case or lower case: ```/search_term\c```
- Search a whole Word: ```/\<search_term\>```
- Search and Replace (the first occurrence) : ```:s/NEMO/NEMO_S/g```
- Search and Replace (everywhere in file) : ```:%s/NEMO/NEMO_S/g```
- Check history commands: ```q/```
- Go to a specific line (e.g. line 60): ```:60``` 
- Search and jump in specific line (e.g. line 120): ```:120 /search_term```
- Go to the next block of lines: ```}```
- Go to the previous block of lines: ```{```

## How to delete fast in vi 

- ```dd``` deletes the whole line under the cursor.
-  ```5dd``` deletes multiple (5) lines, starting at the cursor.
- ```d$``` deletes to the end of the line, starting at the cursor.
- ```dG``` deletes all lines starting from the line under the cursor.

## How to change panels in vimdiff

- ```ctr+w+ctr+h``` : Change cursor to the left panel
-  ```ctr+w+ctr+l``` : change cursor to the right panel
- ```ctr+w+x``` : switch panels
- ```ctr+w+shift+k``` : put the two panels in a top/below format
