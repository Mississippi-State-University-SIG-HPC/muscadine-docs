# Helpful Information

Even if you're new to being on a cluster or even new to Linux as a whole, we still have you covered. We can't teach 
you every Linux command, as there are an infinite number of commands available, but we can show 
you the most common Linux commands and which ones you'll use the most while on the cluster.    

---

## File and Folder Navigation

| Command | Description |
| :-----: | :---------- | 
| pwd | Prints current directory |
| ls | List files or file size with `[-lh]` option |
| cd `<path/to/directory>`| Changes current directory |
| cd `..` | Changes to parent directory |


## Handling Files (Copy, Move, Delete)

| Command | Description |
| :-----: | :---------- |
| mkdir `<name>` | Makes a new directory |
| cp `<file>` `<name>` | Copies file with new name |
| mv `<old path>` `<new path>` | Rename or move files to a new directory |
| rm `<file>` | Removes a file or a folder with `[-r]` option |

```{warning}
Commands that write to a file will almost always overwrite an existing file **without warning**. Always be careful and deliberate with these commands, there is no {kbd}`undo` button.
```

## Reading and Writing to Files

| Command | Description |
| :-----: | :---------- |
| cat `<file>` | Prints output of file in terminal |
| head `<file>` | Shows only top 10 lines of a file |
| tail `<file>` | Shows only the last 10 lines of a file |
| nano `<file>` | Simple text editor |
| vim `<file>` | Highly configurable text editor |

```{tip}
If you've never used vim before, use {kbd}`I` to enter insert mode, {kbd}`Esc` to leave it, {kbd}`:wq` to save and quit, and {kbd}`:q!` to quit without saving.
```

## Helpful Tips

- {kbd}`Tab`: While typing a file or foldername, click tab and linux finish the name for you.
- {kbd}`↑` / {kbd}`↓` : Cycles through your previous commands with the up-and-down arrows.
- {kbd}`Ctrl` + {kbd}`R` : Enters a reverse history search, helpful when a previous command would require pressing  {kbd}`↑` many times.
- {kbd}`Ctrl` + {kbd}`C` : Will kill the current command (Useful if you have a frozen terminal).

---

# More Information

We could have included many more commands on this page that you will eventually use, but what is the point of that? These commands are the foundation for system navigation and file management. Learn these commands first, then learn the more advanced commands when you actually need to use them.

If you do want to dive deeper into more advanced Linux commands, here is a massive and simplified Linux Command Browser: https://tldr.inbrowser.app/
