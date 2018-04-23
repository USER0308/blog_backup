# backup of configuration

## text editor

### Atom

plugins:
* markdown-preview-enhanced
    key: `ctrl + shift + m`  show/hide markdown preview
* markdown-assistant
* minimap
* open-rencent
* pangu
    adding space between Chinese/Japanese/Korean and English characters
* platformmio-ide-terminal
    IDE-like terminal open at current path
    key: ctrl + `
* qiniu-uploader
    upload pictures to qiniu.com

### sublime text
  key: `ctrl + b`  compile
       `ctrl + shift + b` build and run
### vim
plugins:
  vundle: 管理插件
  " 匹配符号插件
	Plugin 'Raimondi/delimitMate'
	" 自动完成插件
	Plugin 'Valloric/YouCompleteMe'
  " 目录文件导航 -----------
	Bundle 'scrooloose/nerdtree'
	" \nt                 打开 nerdree 窗口，在左侧栏显示
	"nmap <leader>nt :NERDTree<CR>
	" 修改打开的快捷键为 F2，关闭的快捷键为 q
	"ctrl+w+h 光标左侧，ctrl+w+l 右侧
	"ctrl + w + w 左右切换窗口
	nmap <F2> :NERDTree<CR>
  for YCM
let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'

" 自动补全配置
set completeopt=longest,menu	" 让 Vim 的补全菜单行为与一般 IDE 一致 (参考 VimTip1228)
autocmd InsertLeave * if pumvisible() == 0|pclose|endif	" 离开插入模式后自动关闭预览窗口
inoremap <expr> <CR>       pumvisible() ? "\<C-y>" : "\<CR>"	" 回车即选中当前项
" 上下左右键的行为 会显示其他信息
inoremap <expr> <Down>     pumvisible() ? "\<C-n>" : "\<Down>"
inoremap <expr> <Up>       pumvisible() ? "\<C-p>" : "\<Up>"
inoremap <expr> <PageDown> pumvisible() ? "\<PageDown>\<C-p>\<C-n>" : "\<PageDown>"
inoremap <expr> <PageUp>   pumvisible() ? "\<PageUp>\<C-p>\<C-n>" : "\<PageUp>"

"youcompleteme  默认 tab  s-tab 和自动补全冲突
"let g:ycm_key_list_select_completion=['<c-n>']
let g:ycm_key_list_select_completion = ['<Down>']
"let g:ycm_key_list_previous_completion=['<c-p>']
let g:ycm_key_list_previous_completion = ['<Up>']
let g:ycm_confirm_extra_conf=0 " 关闭加载. ycm_extra_conf.py 提示

let g:ycm_collect_identifiers_from_tags_files=1	" 开启 YCM 基于标签引擎
let g:ycm_min_num_of_chars_for_completion=2	" 从第 2 个键入字符就开始罗列匹配项
let g:ycm_cache_omnifunc=0	" 禁止缓存匹配项, 每次都重新生成匹配项
let g:ycm_seed_identifiers_with_syntax=1	" 语法关键字补全
nnoremap <F5> :YcmForceCompileAndDiagnostics<CR>	"force recomile with syntastic
"nnoremap <leader>lo :lopen<CR>"open locationlist
"nnoremap <leader>lc :lclose<CR>"close locationlist
inoremap <leader><leader> <C-x><C-o>
" 在注释输入中也能补全
let g:ycm_complete_in_comments = 1
" 在字符串输入中也能补全
let g:ycm_complete_in_strings = 1
" 注释和字符串中的文字也会被收入补全
let g:ycm_collect_identifiers_from_comments_and_strings = 0

nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR> " 跳转到定义处
"---------------------------------------
Bundle 'fholgado/minibufexpl.vim'
" 多文件切换，也可使用鼠标双击相应文件名进行切换
let g:miniBufExplMapWindowNavVim    = 1
let g:miniBufExplMapWindowNavArrows = 1
let g:miniBufExplMapCTabSwitchBufs  = 1
let g:miniBufExplModSelTarget       = 1
" 解决 FileExplorer 窗口变小问题
let g:miniBufExplForceSyntaxEnable = 1
let g:miniBufExplorerMoreThanOne=2
let g:miniBufExplCycleArround=1
" buffer 切换快捷键，默认方向键左右可以切换 buffer
map <F9> :MBEbn<cr>
map <F10> :MBEbp<cr>
"-----------------------------------------
"-------------------------------
" 使用 pyflakes, 速度比 pylint 快
" 语法检查
" <leader>d 快捷键
Bundle 'scrooloose/syntastic'
let g:syntastic_error_symbol = '✗'	"set error or warning signs
let g:syntastic_warning_symbol = '⚠'
let g:syntastic_check_on_open=1
let g:syntastic_enable_highlighting = 0
"let g:syntastic_python_checker="flake8,pyflakes,pep8,pylint"
let g:syntastic_python_checkers=['pyflakes']
"highlight SyntasticErrorSign guifg=white guibg=black

let g:syntastic_cpp_include_dirs = ['/usr/include/']
let g:syntastic_cpp_remove_include_errors = 1
let g:syntastic_cpp_check_header = 1
let g:syntastic_cpp_compiler = 'clang++'
let g:syntastic_cpp_compiler_options = '-std=c++11 -stdlib=libstdc++'
let g:syntastic_enable_balloons = 1	"whether to show balloons

"---------------------------------
"-------------------------
    "-- WinManager setting --
    let g:winManagerWindowLayout='FileExplorer|TagList' " 设置我们要管理的插件
    "let g:persistentBehaviour=0" 如果所有编辑文件都关闭了，退出 vim
    nmap wm :WMToggle<cr>
"---------------------------------------

"------------------------------------------------
    "for C++ 11
nnoremap <F5>   <Esc>:w<CR>:!g++ -std=c++11 % -o /tmp/a.out && /tmp/a.out<CR>
nnoremap <F7>   <Esc>:w<CR>:!g++ -std=c++11 %<CR>
nnoremap <C-F5> <Esc>:w<CR>:!g++ -std=c++11 -g % -o /tmp/a.out && gdb /tmp/a.out<CR>
"-------------------------------------------------
" 代码批量注释
Plugin 'scrooloose/nerdcommenter'

"
"<leader> 前缀，默认为 \
"    <leader>cc 注释当前行
"    <leader>cm 只用一组符号来注释
"    <leader>cy 注释并复制
"    <leader>cs 优美的注释
"    <leader>cu 取消注释


### VS code
paste image to qiniu(需要先apt install xclip)`Ctrl + Alt + v`
vscode-icons
Bracket Pair Colorizer
vscode-fileheader `Ctrl + Alt + i`
key: `Ctrl + B` open/close side bar
`Ctrl + Shift + E` file explorer
`Ctrl + P` open file
`Ctrl + 1` focus on the firsr editor


## IDE

### webstorm
key: `F11` add bookmark at current line
`shift + F11` edit/show the bookmark
`F4` go to the implementing
## browser

### chrome
key: `ctrl + shift + b`  show/hide bookmark
* vimium
key:
```
# Insert your preferred key mappings here.
unmapAll
map j scrollDown
map f scrollUp
map F scrollToTop
map J scrollToBottom
map i focusInput
map G LinkHints.activateMode
map g LinkHints.activateModeToOpenInNewForegroundTab
map q goBack
map p goForward
map n nextTab
map N previousTab
map w removeTab
map W restoreTab
map r reload
map ? showHelp
```
## Application

### Ulancher

#### search
`g` https://google.com/search?q=%s
`so` http://stackoverflow.com/search?q=%s
`wiki` https://en.wikipedia.org/wiki/%s
`baidu` https://www.baidu.com/s?wd=%s
`bing` https://cn.bing.com/search?q=%s
`github` https://github.com/search?q=%s
`csdn` http://so.csdn.net/so/search/s.do?q=%s
`oschina` https://www.oschina.net/search?scope=project&q=%s
`jianshu` https://www.jianshu.com/search?q=%s
`sougouzhihu` http://zhihu.sogou.com/zhihu?query=%s
`zhihu` https://www.zhihu.com/search?type=content&q=%s

#### extension
`bookmark`: google bookmark
`ip`: check my ip
`url`: open in browser
`sys`: system exit/logout
`dict`: translate
`run`: run in shell

### Document Viewer
key:`F9` show/hide side bar

### Diodon
`Ctrl + C`
`Ctrl + V`