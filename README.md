# 
A [codeforces](http://codeforces.com) plugin for vim, ported from vim plugin [gabrielsimoes/cfparser.vim](https://github.com/gabrielsimoes/cfparser.vim).

## Installation
- Install `curl`
- Install `cfparser-pro` with your favorite plugin manager. With `vim-plug`:

```
Plug 'sharifMinhazul/cfparser-pro'
```

### Optional setup
You can setup some variables at your .vimrc:
- `g:cf_cookies_file` - File in which `curl` will store cookies (default: `'~/.cf_cookies'`)
- `g:cf_default_language` - Language to be used when it it not recognized from file extension (default: g:cf_pl_gpp - g++). Languages are mapped in `plugin/cfparser-pro`
- `g:cf_locale` - Language to download problem statement. Either `"ru"` or `"en"` (default: `"en"`)
- You can also redefine the function `cfparser#CFTestAll()`, that is, the function that is called to test your solution against test files. The default definition is as follows. You can redefine the function by writing your own version of it at your `.vimrc`, *after* loading `cfparser-pro`.

```
function! cfparser#CFTestAll() "{{{    
    let match = matchlist(expand('%:p'), s:cf_path_regexp)
    let input = expand('%:p:h'). '/'. match[2]

    if &mod
        execute 'w'
    endif
    if !filereadable(input.'0.in')
        call cfparser#CFDownloadTests()
    endif

    echo system(printf("g++ -Wall -std=c++14 %s -o /tmp/cfparser_exec;
                        \cnt=0;
                        \for i in `ls %s*.in | sed 's/\\.in$//'`; do
                        \   let cnt++;
                        \   echo \"\nTEST $cnt\";
                        \   /tmp/cfparser_exec < $i.in | diff -y - $i.out;
                        \done;
                        \rm /tmp/cfparser_exec",
        \ expand('%:p'), input))
endfunction
```

This will compile the file with `g++` and test it against `a0.in` and `a0.out`, `a1.in` and `a1.out`, etc...

- You also redefine the function `cfparser#CFRun()`, the function that is called to test your solution in an interactive shell. You can redefine it by writing your own version of it at your `.vimrc`, after loading `cfparser-pro`. The default definition is below:

```
function! cfparser#CFRun() "{{{
    if &mod
        execute 'w'
    endif

    echo system(printf("g++ -Wall -std=c++14 %s -o /tmp/cfparser_exec", expand('%s:p')))
        execute '!'. '/tmp/cfparser_exec'
    call system("rm /tmp/cfparser_exec")
endfunction
```

This will compile the file with `g++` and run it with the command warnings and `std=c++14`. You should change `std=c++14` if you want other standard.

## Usage
- `<leader>cfi` - Log**i**n (calls `CFLogin()`)
- `<leader>cfo` - Log**o**ut (calls `CFLogout()`)
- `<leader>cfw` - **W**ho am I (calls `CFWhoAmI()`)
- `<leader>cfp` - Display **p**roblem statement (calls `CFProblemStatement()`)
- `<leader>cfd` - **D**ownload sample tests to current code folder (0.in, 0.out, 1.in ...) (calls `CFDownloadTests()`)
- `<leader>cft` - Runs code with sample **t**ests (calls `CFTestAll()`)
- `<leader>cfr` - *R*uns code in an interactive shell (calls `CFRun()`)
- `<leader>cfs` - **S**ubmit current open file (calls `CFSubmit()`)
- `<leader>cfl` - **L**ist most recent submissions (calls `CFLastSubmissions()` without arguments - can be called with username as argument, default to logged in user)

Submit, download sample tests and problem statement functions "guess" the contest number, problem index and the programming language by the current file name in one of the following forms:
- `directory/505/A/myfile.cpp`
- `directory/505/a.c`
- `directory/505a.c`
