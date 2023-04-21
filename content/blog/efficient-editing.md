---
layout: post
title: Efficient Editing with Vim
date: 2006-05-03
---

> "To me, vi is Zen. 
> To use vi is to practice zen.
> Every command is a koan.
> Profound to the user,
> unintelligible to the uninitiated.
> You discover truth every time you use it."

*--reddy@lion.austin.com*

This tutorial assumes a basic knowledge of vim -- insert mode, command mode, loading and saving files, etc. It is intended to help vi novices develop their skills so that they can use vi efficiently.

In this tutorial, `<C-X>` means Ctrl-X -- that is, hold down the `Ctrl` key and press `X`. You can get help on most of the commands used here by typing `:help command` in vim, where command is what you need help on.

## Moving efficiently

### Stay out of insert mode

In general, you want to spend as little of your time in vim's insert mode as possible, because in that mode it acts like a dumb editor. This is why most vim novices spend so much time in insert mode -- it makes vim easy to use. But vim's real power lies in command mode! You'll find that the better you know vim, the less time you will spend in insert mode.

### Use h, j, k, and l

The first step to efficient editing in vim is to wean yourself from the arrow keys. One of the the advantages of vim's modal design is that you do not need to constantly move your hands back and forth between the arrow keys and the letter keys; when you are in command mode, the letters `h`, `j`, `k` and `l` correspond to the directions left, down, up, and right, respectively. It takes some practice to get used to, but you will notice the speed difference once you're used to it.

When you are editing e-mail or other paragraph-formatted text, you might notice that the direction keys skip more lines than you expect. This is because your paragraphs appear as one long line to vim. Type `g` before `h`, `j`, `k` or `l` to move by screen lines instead of virtual lines.

### Use motions to move the cursor in the current line

Most editors have only simple commands for moving the cursor (left, up, right, down, to beginning/end of line, etc). vim has very advanced commands for moving the cursor; these commands are referred to as motions. When the cursor moves from one point in the text to another, the text between the points (and including the points themselves) is considered to be "moved over." (This will be important later.)

Here are a few of the more useful motions:

| `fx` | Move the cursor forward to the next occurance of the character x on the current line (obviously, x can be any character you like). This is an extremely useful command. You can type ; to repeat the last f command you gave.
| `tx` | Same as above, but moves the cursor to right before the character, not all the way to it. (It's very useful, really.)
| `Fx` | Move the cursor backward to the next occurance of the character x on the current line.
| `w`  | Move the cursor forward by a word.
| `b`  | Move the cursor backward by a word.
| `0`  | Move the cursor to the beginning of the current line.
| `^`  | Move the cursor to the first character on the current line.
| `$`  | Move the cursor to the end of the line
| `)`  | Move the cursor forward to the next sentence. (Useful when editing e-mail or text documents.)
| `(`  | Move the cursor backward by a sentence.


### Move efficiently through the file

vim has many commands that can send you to where you want to go in your file -- there's rarely a need to scroll manually through it. The below keystrokes are not technically motions, since they move around in the file instead of in a particular line.

| `<C-F>`	| Move the cursor forward by a screenful of text
| `<C-B>`	| Move the cursor backward by a screenful of text
| `G`	    | Move the cursor to the end of the file
| `numG`	| Move the cursor line num. (For instance, 10G moves to line 10.)
| `gg`  	| Move the cursor to the beginning of the file
| `H`   	| Move the cursor to the top of the screen.
| `M`	    | Move the cursor to the middle of the screen.
| `L`	    | Move the cursor to the bottom of the screen.
| `*`	    | Read the string under the cursor and go to the next place it appears. (For instance, if your cursor was somewhere on the word "bob," the cursor would move to the next occurance of "bob" in your file.)
| `#`	    | Same as above, except it moves the cursor to the previous occurance.
| `/text`	| Starting from the cursor, find the next occurance of the string text and go to it. You will need to press Enter to execute the search. To re-execute your last search, type n (for next occurance).
| `?text`	| Same as /, but searches in the opposite direction.
| `ma`	    | Make a bookmark named *a* at the current cursor position. A bookmark can be named any lowercase letter. You can't see the bookmark, but it's there!
| `` `a``	| Go to bookmark *a*. Important: that's a backtick, not a single quote. The backtick is located to the left of the 1 on most keyboards.
| `` `.``	| Go to the line that you last edited. This is very useful! If you need to scroll through the file to look something up, you can go back to where you were without bookmarking it by using the `` `.`` command.


## Typing efficiently

### Use keyword completion

vim has a very nice keyword completion system. This means that you can type part of a long word, press a key, and have vim finish the word for you. For instance, if you have a variable called *iAmALongAndAwkwardVarName* somewhere in your code, you probably don't want to type the whole thing in every time you use it.

To use keyword completion, just type the first few letters of the string (e.g. `iAmAL`) and press `<C-N>` (that means hold down Ctrl and type N) or `<C-P>`. If vim doesn't give you the word you want at first, keep trying -- vim will cycle through all completions it can find.

### Enter insert mode intelligently

Most users new to vim get into insert mode by typing `i`. This works, but it's often pretty inefficient, since vi has a host of commands that leave the editor in insert mode. Here are some of the more popular ones:

| `i`	        | Insert text to the left of the current character.
| `I`	        | Insert text at the beginning of the current line.
| `a`	        | Insert text to the right of the current character.
| `A`	        | Insert text at the end of the current line.
| `o`	        | Create a new line under the current one and insert text there.
| `O`	        | Create a new line above the current one and insert text there.
| `c{motion}`	| Delete (change) the text moved over by {motion} and insert text to replace it. For instance, `c$` would delete the text from the cursor to the end of the line and enter insert mode. `ct!` would delete the text from the cursor up to (but not including) the next exclamation mark and enter insert mode. The deleted text is copied to the clipboard and can be pasted.
| `d{motion}`	| Delete the text moved over by {motion} -- same as `c{motion}`, but doesn't enter insert mode.


## Moving blocks of text efficiently

### Use visual selections and the appropriate selection mode

Unlike the original vi, vim allows you to highlight text and perform operations on it. There are three main visual selection modes (that is, text highlighting modes). These modes are as follows:

| `v`	  | Characterwise selection mode. This is the selection mode that most people are used to, so practice with it before trying the others.
| `V`     |	Linewise selection mode. Whole lines are always selected. This is better than characterwise mode when you want to copy or move a group of lines.
| `<C-V>` |	Blockwise selection mode. Extremely powerful and available in very few other editors. You can select a rectangular block and any text inside that block will be highlighted.

All the usual cusor movement keys apply -- so, for instance, `vwww` would go into visual selection mode and highlight the next three words. `Vjj` would go into linewise visual selection mode and highlight the current line and the two lines below it.

### Cutting and copying from visual selections

Once you have a highlighted selection, you probably want to do something with it. Some of the more useful commands you can give when an area of text is highlighted:

| `d`	| Cut (delete) the highlighted text and put it into the clipboard.
| `y`	| Copy (or yank, which is vim-ese for "copy") the highlighted text into the clipboard.
| `c`	| Cut the highlighted text into the clipboard. This is just like d, except it leaves the editor in insert mode.

### Cutting and copying from non-visual selections

If you know exactly what you want to copy or cut, you can do it without entering visual mode. This saves time.

| `d{motion}`	| Cut the text moved over by {motion} to the clipboard. For instance, dw would cut a word and dfS would cut from the cursor up to and including the next capital S on the current line of text.
| `y{motion}`	| Copy the text moved over by {motion}.
| `c{motion}`	| Cut the text moved over by {motion} and leave the editor in insert mode.
| `dd`	        | Cut the current line.
| `yy`	        | Copy the current line.
| `cc`	        | Cut the current line and leave the editor in insert mode.
| `D`	        | Cut from the cursor to the end of the current line.
| `Y`	        | Yank the whole line, just like yy. (Yes, it's inconsistent! You can use y$ to do what you would expect Y to do.)
| `C`	        | Cut from the cursor to the end of the current line and leave the editor in insert mode.
| `x`	        | Cut the current character. (This is sort of like a command-mode backspace.)
| `s`	        | Cut the current character and leave the editor in insert mode.

### Pasting

Pasting is easy. Put the cursor where you want the pasted text and type `p`.

### Using multiple clipboards

Most editors have a single clipboard. vim has many more; clipboards in vim are called registers. You can list all the currently defined registers and their contents by typing `:reg`. Typically, you'll be using the lowercase letter registers; the others are used for various internal vim purposes and are only occasionally helpful.

To use a specific register for a copy or paste operation, simply type `"a` before the command for the operation, where `a` is the register you want to use.

For example, to copy the current line into register k, you could type `"kyy`. (You could also type `V"ky`. Why would that work?). That line would stay in register *k* until you specifically copied something else into register *k*. You would then use `"kp` to paste the text from register *k*.

## Avoiding repetition

### The amazing . command

In vi, typing `.` (a period) will repeat the last command you gave. For instance, if your last command was `dw` (delete word), vi will delete another word.

### Using counts

Counts are one of the most powerful and time-saving features of vim. Any command can be preceded by a number. The number will tell vim how many times to execute the command. Here are a few examples:

`3j` will move the cursor down three lines.

`10dd` will delete ten lines.

`y3"e;` will yank (copy) text from the cursor to the third quotation mark after the cursor on the current line. Counts are useful to extend the range of a motion in this manner.

### Recording macros

Occasionally, you'll find yourself doing the same thing over and over to blocks of text in your document. vim will let you record an ad-hoc macro to perform the operation.

| `qregister`	| Start macro recording into the named register. For instance, `qa` starts recording and puts the macro into register *a*.
| `q`	        | End recording.
| `@register`	| Replay the macro stored in the named register. For instance, `@a` replays the macro in register *a*.

Keep in mind that macros just record your keystrokes and play them back; they are not magic. Recording macros is almost an art form because there are so many commands that accomplish a given task in vim, and you must carefully select the commands you use while your macro is recording so that they will work in all the places you plan to execute the macro.

## Writing code in vim

vim is an excellent editor for source code because it has many features that are specifically designed to help programmers. Here are a few of the more handy ones:

| `]p`	| Just like p, but it automatically adjusts the indent level of the pasted code to match that of the code you paste into. Try it!
| `%`   | Putting the cursor on a brace, bracket, or parenthese and pressing % will send the cursor to the matching brace, bracket, or parenthese. Great for fixing parse problems related to heavily nested blocks of code or logic.
| `>>`	| Indent the highlighted code. (See the earlier section about efficient text selection. If no text is selected, the current line is indented.)
| `<<`	| Like >>, but un-indents.
| `gd`	| Go to the definition (or declaration) of the function or variable under the cursor.
| `K`   | Go to the man page for the word currently under the cursor. (For instance, if your cursor is currently over the word sleep, you will see the man page for sleep displayed.)



