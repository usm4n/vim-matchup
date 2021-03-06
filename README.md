# vim match-up

:warning: warning :warning: this plugin is under heavy
active development.  There are probably bugs!

match-up is a replacement for the venerable vim plugin [matchit.vim].
match-up aims to replicate all of matchit's features, fix a number of its
deficiencies and bugs, and add a few totally new features.  It also
replaces the standard plugin [matchparen], allowing all of matchit's words
to be highlighted along with the `matchpairs` (`(){}[]`).

[matchit.vim]: http://ftp.vim.org/pub/vim/runtime/macros/matchit.txt
[matchparen]: http://ftp.vim.org/pub/vim/runtime/doc/pi_paren.txt

<img src='https://github.com/andymass/matchup.vim/wiki/images/teaser.jpg' width='300px' alt='and in this corner...'>

A major goal of this project is to keep a modern and modular code base.
Contributions are welcome!

## Table of contents

  * [Overview](#overview)
  * [Installation](#installation)
  * [Features](#features)
  * [Options](#options)
  * [FAQ](#faq)
  * [Interoperability](#interoperability)
  * [Acknowledgments](#acknowledgments)
  * [Development](#development)

## Overview

This plugin

- Extends vim's `%` motion to language-words like `if`, `else`, `endif`.
- Adds motions `g%`, `[%`, `]%`, and `z%`.
- Combines these motions into convenient text objects `i%` and `a%`.
- Highlights symbols and words under the cursor which `%` can work on,
  and highlights matching symbols and words.  Now you can easily tell
  where `%` will jump to.

Planned features:

- Add auto-completion for words and symbols- for example you could 
  automatically insert corresponding a `)` or `endif`.

## Installation

If you use vim-plug, then add the following line to your vimrc file:

```vim
Plug 'andymass/matchup.vim'
```

Or use some other plugin manager:

  - vundle
  - neobundle
  - pathogen

## Features

|         | feature                          | __match-up__   | matchit       | matchparen    |
| ------- | -------------------------------- | -------------- | ------------- | ------------- |
| ([a.1]) | jump between matching words      | :thumbsup:     | :thumbsup:    | :x:           |
| ([a.2]) | jump to open & close words       | :thumbsup:     | :question:    | :x:           |
| ([a.3]) | jump inside                      | :thumbsup:     | :x:           | :x:           |
| ([b.1]) | full set of text objects         | :thumbsup:     | :question:    | :x:           |
| ([c.1]) | highlight `()`, `[]`, & `{}`     | :thumbsup:     | :x:           | :thumbsup:    |
| ([c.2]) | highlight _all_ matches          | :thumbsup:     | :x:           | :x:           |
| ([c.3]) | display matches off-screen       | :thumbsup:     | :x:           | :x:           |
| ([d.1]) | parallel transmutation           | :construction: | :x:           | :x:           |
| ([e.1]) | modern, modular coding style     | :thumbsup:     | :x:           | :x:           |
| ([e.2]) | actively developed               | :thumbsup:     | :x:           | :x:           |

[a.1]: #a1-jump-between-matching-words
[a.2]: #a2-jump-to-open-and-close-words
[a.3]: #a3-jump-inside
[b.1]: #b1-full-set-of-text-objects
[c.1]: #c1-highlight---and-
[c.2]: #c2-highlight-all-matches
[c.3]: #c3-display-matches-off-screen
[d.1]: #d1-parallel-transmutation
[e.1]: #development
[e.2]: #development
[inclusive]: #inclusive-and-exclusive-motions
[exclusive]: #inclusive-and-exclusive-motions

Legend: :thumbsup: supported. :construction: TODO, planned, or in progress.
:question: poorly implemented, broken, or uncertain.  :x: not possible.

### Detailed feature documentation

What do we mean by open, close, mid?  This depends on the specific file
type and is configured through the variable `b:match_words`.  Here are a
couple examples:

#### vim-script

```vim
if l:x == 1
  call one()
else
  call two()
elseif
  call three()
endif
```

For the vim-script language, match-up understands the words `if`,
`else`, `elseif`, `endif` and that they form a sequential construct.  The
"open" word is `if`, the "close" word is `endif`, and the "mid"
words are `else` and `elseif`.  The `if`/`endif` pair is called an
"open-to-close" block and the `if`/`else`, `else`/`elsif`, and
`elseif`/`endif` are called "any" blocks.

#### C, C++
```c
#if 0
#else
#endif

void some_func() {
    if (true) {
      one();
    } else if (false && false) {
      two();
    } else {
      three();
    }
}
```

Since in C and C++, blocks are delimited using braces (`{` & `}`),
match-up will recognize `{` as the open word and `}` as the close word.
It will ignore the `if` and `else if` because they are not defined in
vim's C file type plugin.

On the other hand, match-up will recognize the `#if`, `#else`, `#endif`
preprocessor directives.

#### (a.1) jump between matching words
  - `%` go forwards to next matching word.  If at a close word,
  cycle back to the corresponding open word.
  - `{count}%` forwards `{count}` times.  Requires
  `{count} <= g:matchup_motion_override_Npercent`.  For larger
  `{count}`, `{count}%` goes to the `{count}` percentage in the file.
  - `g%` go backwards to `[count]`th previous matching word.  If at an
  open word, cycle around to the corresponding close word.

#### (a.2) jump to open and close words
- `[%` go to `[count]`th previous outer open word.  Allows navigation
to the start of blocks surrounding the cursor.  This is similar to vim's
built-in `[(` and `[{` and is an [exclusive] motion.
- `]%` go to `[count]`th next surrounding close word.  This is an
[exclusive] motion.

#### (a.3) jump inside
- `z%` go to inside `[count]`th nearest inner contained block.  This
  is an [exclusive] motion when used with operators, except it eats
  whitespace.  For example, where `█` is the cursor position,

```vim
  █ call somefunction(param1, param2)
```
`dz%` produces
```vim
  param1, param2)
```
but in
```vim
  █ call somefunction(      param1, param2)
```
`dz%` also produces
```vim
  param1, param2)
```

#### (b.1) full set of text objects
- `i%` the inside of an any block
- `1i%` the inside of an open-to-close block
- `{count}i%` If count is greater than 1, the inside of the `{count}`th
  surrounding open-to-close block

- `a%` an any block.
- `1a%` an open-to-close block.  Includes mids but does not include open
  and close words.
- `{count}a%` if `{count}` is greater than 1, the `{count}`th surrounding
  open-to-close block.

See [here](#line-wise-operatortext-object-combinations)
for some examples and important special cases.

#### (c.1) highlight `()`, `[]`, and `{}`

match-up emulates vim's matchparen to highlight the symbols contained
in the `matchpairs` setting.

#### (c.2) highlight _all_ matches          

To disable match highlighting, `let g:matchup_matchparen_enabled = 0`.
If this option is set before the plugin is loaded, it will not disable
the built-in matchparen plugin.  See [here](#module-matchparen) for
other related options.

#### (c.3) display matches off screen

If a open or close which would have been highlighted is on a line
positioned outside the current window, the match is shown in the
status line.  If both the open and close match are off-screen, the
close match is preferred.

#### (d.1) parallel transmutation

In insert mode, after changing text inside a word, matching words will
be changed in parallel.  As an example,

```html
<pre>
  text
</pre>
```

Changing `pre` to `div` and leaving insert mode will produce:

```html
<div>
  text
</div>
```

Note: this currently only works for match words which define a backref
relation like `\1`.  A wider set of transmutations are planned.

Parallel transmutation requires the matchparen module to be enabled.

_Planned_: `g:matchup_transmute_auto`, `CTRL-G %` mapping.  A
corresponding normal mode command is also planned.

### Inclusive and exclusive motions

In vim, character motions following operators (such as `d` for delete
and `c` for change) are either _inclusive_ or _exclusive_.  This means
they either include the ending position or not.  match-up is designed so
that `d]%` inside a set of parenthesis behaves exactly like `d])`.  For
other words, exclusive motions will not include the close word.  In this
example, where `█` is the cursor position,

```vim
if █x | continue | endif
```

pressing `d]%` will produce    

```vim
if endif
```

To include the close word, use either `dv]%` or `vd]%`.  This is vim
compatible with `d])` and `d]}`.

Unlike `]%`, `%` is an _inclusive_ motion.  As a special case for the 
`d` (delete) operator, if `d%` leaves behind lines white-space, they will
be deleted also.  In effect, it will be operating line-wise.  As an
example, pressing `d%` will leave behind nothing.

```text
   █(

   )
```

To operate character-wise in this situation, use `dv%` or `vd%`.
This is vim compatible with the built-in `d%` on `matchpairs`.

### Line-wise operator/text-object combinations

Normally, the text objects `i%` and `a%` work character-wise.  However,
there are some special cases.  For certain operators combined with `i%`,
under certain conditions, match-up will effectively operate line-wise
instead.  For example, in
```vim
if condition
 █call one()
  call two()
endif
```
pressing `di%` will produce
```vim
if condition
endif
```
even though deleting ` condition` would be suggested by the object `i%`.
The intention is to make operators more useful in some cases.  The
following rules apply:
1. The operator must be listed in `g:matchup_text_obj_linewise_operators`.
  By default this is `d` and `y` (e.g., `di%` and `ya%`).
2. The outer block must span multiple lines.
3. The open and close delimiters must be more than one character long.  In
  particular, `di%` involving a `(`...`)` block will not be subject to
  these special rules.

To prevent this behavior for a particular sequence `dvi%` or `vdi%`.

To disable this entirely, remove the operator from the following variable,
```vim
let g:matchup_text_obj_linewise_operators = [ 'y' ]
```

Note: unlike vim's built-in `i)`, `ab`, etc., `i%` does not make an
existing visual mode character-wise.

A second special case involves `da%`.  In this example,
```vim
    if condition
     █call one()
      call two()
    endif
```
pressing `da%` will delete all four lines and leave no white-space.  This
is vim compatible with `da(`, `dab`, etc.

## Options

To disable the plugin entirely,
```vim
let g:matchup_enabled = 0
```
default: 1

To disable a particular module,
```vim
let g:matchup_matchparen_enabled = 0
let g:matchup_motion_enabled = 0
let g:matchup_text_obj_enabled = 0
```
defaults: 1

To enable the experimental [transmute](#d1-parallel-transmutation)
module,
```vim
let g:matchup_transmute_enabled = 1
```
default: 0

### Variables  

matchup understands the following variables from matchit.
- `b:match_words`
- `b:match_skip`
- `b:match_ignorecase`

These are set in the respective ftplugin files.  They may not exist for
every file type.  To support a new file type, create a file
`after/ftplugin/{filetype}.vim` which sets them appropriately.

### Module matchparen

The matchparen module can be disabled on a per-buffer basis

```vim
let b:matchup_matchparen_enabled = 0
```
default: 1

If this module is disabled on a particular buffer, match-up will still
fall-back to the vim standard plugin matchit, which will highlight
`matchpairs` such as `()`, `[]`, & `{}`.  To disable this,
```vim
let b:matchup_matchparen_fallback = 0
```
default: 1

A common usage is to automatically disable matchparen for
particular file types;
```vim
augroup matchup_matchparen_disable_ft
  autocmd!
  autocmd FileType tex let [b:matchup_matchparen_fallback,
      \ b:matchup_matchparen_enabled] = [0, 0]
augroup END
```

Whether to highlight known words even if there is no match:
```vim
let g:matchup_matchparen_singleton = 1
```
default: 0

Whether to replace the statusline for off-screen matches:
```vim
let g:matchup_matchparen_status_offscreen = 0
```
default: 1

Adjust timeouts in milliseconds for matchparen highlighting
```
let g:matchup_matchparen_timeout = 300
let g:matchparen_insert_timeout = 60
```
default: 300, 60

### Module motion

In vim, `{count}%` goes to the `{count}` percentage in the file.
match-up overrides this motion for small `{count}` (by default, anything
less than 7).  To allow `{count}%` for `{count}` up to 11,
```vim
g:matchup_motion_override_Npercent = 11
```
To disable this feature, and restore vim's default `{count}%`,
```vim
g:matchup_motion_override_Npercent = 0
```
default: 6

If enabled, cursor will land on the end of mid and close words while
moving downwards (`%`/`]%`).  While moving upwards (`g%`, `[%`) the cursor
will land on the beginning.  To disable,
```vim
let g:matchup_motion_cursor_end = 0
```
default: 1

### Module text_obj

Modify the set of operators which may operate
[line-wise](#line-wise-operatortext-object-combinations)
```vim 
let g:matchup_text_obj_linewise_operators' = ['d', 'y']
```
default: `['d', 'y']`

### Module transmute

_Options planned_.

## FAQ

- match-up doesn't work

  The plugin requires a fairly recent version of vim.  Please tell me your
  vim version and error messages.  Try updating vim and see if the problem
  persists.

- Why does jumping not work for construct X in language Y?

  Please open a new issue 

- Highlighting is not correct for construct X

  match-up uses matchit's filetype-specific data, which may not give
  enough information to create proper highlights.  To fix this, you may
  need to modify `b:match_words`.

  For help, please open a new issue and be a specific as possible.

- I'm having performance problems

  match-up aims to be as fast as possible.  If you see any performance
  issues, please open a new issue and report `g:matchup#perf#times`.

- How can I contribute?

  Read the [contribution guidelines](CONTRIBUTING.md) and [issue
  template](ISSUE_TEMPLATE.md).  Be as precise and detailed as possible
  when submitting issues and pull requests.

## Interoperability

  - match-up's match highlighting is not compatible with [vimtex]'s
  implementation.  match-up highlighting and will be disabled
  automatically when vimtex is detected.
  - matchit.vim should not be loaded.  If it is loaded, it must be loaded
  before match-up (in this case, matchit will be disabled when match-up
  loads).
  - match-up loads matchparen if it is not already loaded.

## Acknowledgments 

### Origins

match-up was originally based on [@lervag](https://github.com/lervag)'s
[vimtex].  The concept and style of this plugin and its development are
heavily influenced by vimtex. :beers:

[vimtex]: https://github.com/lervag/vimtex

### Other inspirations

- [matchit](http://ftp.vim.org/pub/vim/runtime/macros/matchit.txt)
- [matchparen](http://ftp.vim.org/pub/vim/runtime/doc/pi_paren.txt)
- [MatchTagAlways](https://github.com/Valloric/MatchTagAlways)
- [vim-endwise](https://github.com/tpope/vim-endwise)
- [auto-pairs](https://github.com/jiangmiao/auto-pairs)
- [delimitMate](https://github.com/Raimondi/delimitMate)
- [splitjoin.vim](https://github.com/AndrewRadev/splitjoin.vim)
- [vim-surround](https://github.com/tpope/vim-surround)
- [vim-sandwich](https://github.com/machakann/vim-sandwich)


## Development

### Reporting problems

This is a brand new plugin and there are likely to be many bugs.
Thorough issue reports are encouraged.  Please read the [issue
template](ISSUE_TEMPLATE.md) first.  Be as precise and detailed as
possible when submitting issues.  

Feature requests are also welcome.

### Contributing

Please read the [contribution guidelines](CONTRIBUTING.md) before
contributing.

### Planned feature wish-list

This is a set of features planned for "version 1" but require a bit more
research before they can be properly tackled.

|         | feature                          | __match-up__   | matchit       | matchparen    |
| ------- | -------------------------------- | -------------- | ------------- | ------------- |
| ([d.2]) | auto-insert open, close, & mid   | :construction: | :x:           | :x:           |
| ([d.3]) | completion                       | :construction: | :x:           | :x:           |
| ([d.4]) | split & join                     | :construction: | :x:           | :x:           |

[d.2]: #d2-auto-insert-open-close-and-mid
[d.3]: #d3-completion
[d.4]: #d4-split-and-join

#### (d.2) auto-insert open, close, and mid

_Planned_.

- end-wise style completion: typing `CTRL-X <cr>` will insert the 
corresponding end word (mapping subject to change).

- automatic block insertion: typing `CTRL-X CTRL-B` to produce
block skeletons (mapping subject to change).

#### (d.3) completion

_Planned_. 

Typing `CTRL-X O`/`CTRL-X W` will give a menu of possible words (mapping
subject to change).

#### (d.4) split and join

_Planned_.

Convert between single-line and multi-line blocks.  Mappings undecided.

### Todo list

- write proper vim doc
- thoroughly test with unicode, tabs
- complete parallel transmutation in an efficient way.
- add screenshots and animations
- support for fenced code possible?
- add file type `quirks` module
- investigate whether `&selection`/`&virtualedit` options are important
- can match-up be integrated with
  [vim-surround](https://github.com/tpope/vim-surround)?
- make sure `b:match_ignorecase` works
- support python (like python_match.vim)

