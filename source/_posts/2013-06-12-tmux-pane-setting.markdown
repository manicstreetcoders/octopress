---
layout: post
title: "Tmux pane setting"
date: 2013-06-12 20:02
comments: true
categories: 
---

tmux 와 vimrc 에 몇가지를 추가했는데.

### tmux

pane 을 vi key 로 옮겨다니기 위해서

<pre>
unbind j
unbind k
unbind h
unbind l
bind j select-pane -D
bind k select-pane -U
bind h select-pane -L
bind l select-pane -R
</pre>

를 해서 자판에서 손을 안떼고 움직일 수 있게 하였다.

그리고,

<pre>
bind / command-prompt "split-window 'exec %%'"
</pre>

를 추가해서 간단한 shell command 를 tmux 안에서 날릴 수 있게 하였다.
 
C-a / 를 누르고 ping localhost 를 때리면, pane 이 하나 열리고 ping 이 실행된다.

    bind a send-prefix

로 Ctrl-a a 로 Ctrl-a 기능을 살린다. Ctrl-a / Ctrl-e / Ctrl-b 등의 Emacs 스타일 cursor moving 을 자주 쓴다면 필요.

그리고 tmux 가 최근에 1.8 로 업글되었다. 
Ubuntu 의 경우 패키지로 아직 repo 에 안올라와서, source 를 컴파일해야 한다. 
OS X 의 경우 brew 에 1.8 패키지가 올라왔다. `brew install tmux` 또는 `brew upgrade tmux` 하면 된다.
1.8 의 큰 장점은 pane zoom 이 된다는 것. 그 전까지는 꽁수로 했었는데, 이제는 native 하게 된다. 
`Ctrl-a z` 로 zoom 을 토글할 수 있다. 줌기능 대박이다.

Ubuntu 에서 tmux 카피 버퍼와 OS 클립보드를 통합하는 방법:

    $ sudo apt-get install xclip

    bind C-c run "tmux save-buffer - |xclip -i -sel clipboard"
    bind C-v run "tmux set-buffer \"$(xclip -o -sel clipboard)\"; tmux paste-buffer"

OS X 에서 통합하는 방법:

    $ git clone https://github.com/ChrisJohnsen/tmux-MacOSX-pasteboard.git
    $ cd tmux-MacOSX-pasteboard
    $ make reattach-to-user-namespace
    $ mv reattach-to-user-namespace /usr/local/bin
    $ export PATH="/usr/local/bin:$PATH"

    set -g default-command "reattach-to-user-namespace -l /bin/zsh"
    bind C-c run "tmux save-buffer - | reattach-to-user-namespace pbcopy"
    bind C-v run "tmux set-buffer $(reattach-to-user-namespace pbpaste); tmux paste-buffer"

이제 C-a C-c 로 tmux buffer -> OS X clipboard 로 카피할 수 있다.

### vim

만약 Solarized 를 안쓴다면, TabLine 을 바꾸는 것이 좋다.

.vimrc 에

<pre>
hi TabLineSel ctermfg=Red ctermbg=Yellow
hi TabLine ctermfg=Blue ctermbg=Yellow
hi TabLineFill ctermfg=LightGreen ctermbg=DarkGreen
</pre>

ftplugin 으로 안하고, 그냥 .vimrc 에 때려박은 것들이 있는데,

    autocmd FileType python set sts=4|set sw=4|set expandtab
    autocmd FileType ruby set sts=2|set sw=2|set expandtab

나중에 filetype plugin 으로 정리할 생각.

사람들이 잘 모르는 것이 sts 와 ts 와 sw 의 차이.

expandtab 을 할 경우에는 ts 를 할 필요없이 sts 만 해줘도 된다.

그리고

<pre>
noremap &lt;CR> :
</pre>

를 했는데.

ESC : 로 커맨드모드를 진입하면 : 를 누를때 SHIFT 를 같이 눌러야해서,
손이 아팠는데, 이제는 ESC ENTER 로 커맨드모드 진입이 가능해진다.

손이 한결 편해진다.

#### 10 J

그리고 vim 에서 흔히 텐제이라고 부르는 매핑.

<pre>
map &lt;C-j> 10j
map &lt;C-k> 10k
</pre>

로 빠르게 상하이동이 가능하다. Ctrl + J / Ctrl + K

#### Indentation

Code indentation 은 visual mode 에서 &lt; &lt; 로 할 수 있는데, 좀 더 편하게 하기 위해서
visual mode non-recursive mapping 을 추가한다.

    vnoremap < <gv
    vnoremap > >gv

이렇게 하면, Visual mode 에서 &lt; 를 연속으로 눌러 indentation 을 들이거나 낼 수 있다.
파이썬같이 인덴테이션이 중요한 언어에서는 꼭 필요한 매핑이다.

vim 팁으로는,

코드 인덴팅할때, 편리한 vim 커맨드.

<pre>
5&lt;&lt;
Vjj&lt;
&lt;%
]p
</pre>

#### .vimrc

    nnoremap <leader>ev :vsplit $MYVIMRC<cr>

하면 편하게 .vimrc 를 에디팅할 수 있다.

#### Rails

Rails 를 사용한다면, route 화일이나 DB 스키마를 에디트할 경우가 많다. 그래서 다음 매크로가 유용하다.

    command! Rroutes :e config/routes.rb
    command! Rschema :e db/schema.rb

#### Vim Plugin

vim plugin 중에서는 Command-T, Solarized, YouCompleteMe, Taglist 를 사용.

Taglist 를 쓸 경우, .vimrc 에 (OS X 에서 brew 로 Exuberant ctags 를 깔았을 경우)

    nnoremap <silent> <F8> :TlistToggle<CR>
    let Tlist_Ctags_Cmd='/usr/local/Cellar/ctags/5.8/bin/ctags'

<F8> 로 클래스/함수 pane 을 on/off 시킬 수 있다.

YouCompleteMe 때문에, OS X 에서는 MacVim 에 딸려오는 Vim 을 사용한다 (nogui). 만약 SIGSEGV 가 발생하면, 아마 python library 가 잘못 링크되었을 경우인데, 임시로 brew unlink python 으로 python 라이브러리 쫑나는 것 해결할 수 있다.

MacVim 은 [여기서](http://github.com/b4winckler/macvim/downloads) 다운받아 컴파일한다.

Ubuntu 에서는 ruby 와 python 써포트를 On 하고, 링크되는 라이브러리에 신경을 써서 새로 컴파일하는 것이 좋다. 12.04 에 딸려오는 vim 이 너무 구식이라.

OS X iTerm2 터미널도 Solarize 할 수 있는 스크립이 찾아보면 있으니. [https://github.com/altercation/solarized](altercation/solarized/iterm2-colors-solarized)

그리고, Linux gnome 의 경우도 github 에 [https://github.com/sigurdga/gnome-terminal-colors-solarized](sigurdga/gnome-terminal-colors-solarized) 로 찾으면 나옴.

vim colorscheme 만 solarized 를 쓰면 terminal 로 돌아올때 번쩍해서 눈이 피곤하다. 둘 다 맞춰주는 것이 좋다.

Command-T 를 설치할 때, Linux 는 간단한데, OS X 는 약간 까다롭다.

$ vim --version 으로 vim 컴파일에 사용된 FLAG 랑 맞춰주는 것이 기본인데,
OS X Mountain Lion 에서는
처음에 아무생각없이 build 했다가 SIGSEGV 이 발생해서,

<pre>
vim --version|grep arch
vim --version|grep ruby
ARCHFLAGS="-arch x86_64"
rbenv local system 
ruby --version
ruby extconf.rb
make
</pre>

rbenv 로 /usr/bin/ruby 가 실행되게 하는 것.
ruby 버전이 1.8.7 인지 확인하는 것.
vim 이 컴파일된 아키텍쳐와 루비 써포트를 확인하는 것.

vim 이 +ruby 가 아니라면, [Installing vim with ruby support (+ruby)](http://stackoverflow.com/questions/3794895/installing-vim-with-ruby-support-ruby) 참조.


15 인치 노트북에서 화면 가득한 iTerm 을 띄워놓고,
vim window + vim tab + tmux pane + tmux screen 을 활용하면,
마우스나 트랙패드를 사용할 필요가 없다.

iTerm 과 크롬을 한개씩 띄워놓고 개발이 가능해진다.

CMD-TAB 과 크롬 단축기를 사용하면, 키보드에서 손을 뗄 필요가 없어진다.

참고로 iTerm 의 경우 CMD+OPTION+1 등으로 윈도우를 선택할 수 있다.

#### Vim Key Mapping

그리고 vim key mapping 에 관한 좋은 글 
Mapping keys in Vim - Tutorial - 
<a href="http://vim.wikia.com/wiki/Mapping_keys_in_Vim_-_Tutorial_(Part_1)">here</a>

vim 으로 hex edit 하는 법

    :%!xxd
    :%!xxd -r

다량의 Tab 을 열어놓고 작업하다가 한꺼번에 닫고 나갈때,

    :qa
    :wqa

하면 된다.

### irb

ruby 를 사용하면 irb 를 많이 쓰게되는데, irb 에서 vi key 를 쓰는 방법.

    $ vim ~/.editrc
    bind -v
    bind \\t rl_complete

이렇게 하면 ESC i j k l 등으로 이동할 수 있고, TAB 으로 completion 을 할 수 있다.

python REPL 에서도 먹는다. Emacs 스타일보다 Vim 스타일이 더 편하다.

### ctags & cscope

Python 코딩은 OS X 보다는 Linux 에서 작업하는데,

    sudo apt-get install ctags 

~/.ctags 에다가

    --python-kinds=-i

그리고 .vimrc 에다가

    set tags=./mytags

그리고 

    ctags -R -o ./mytags ~/code

를 해두면,

vim 에서,

    vim -t <tag name>
    Ctrl-] / Ctrl-T
    :tselect / :stselect
    :tnext / :tprev
    :help tags

으로 tag 들을 navigating 할 수 있다.

그리고 ctags v.s. cscope 에 관한 좋은 글 [cscope or ctags why choose one over the other?](http://stackoverflow.com/questions/934233/cscope-or-ctags-why-choose-one-over-the-other)

그리고, OS X 에서 Exuberant ctags 를 사용해서 python 등을 코딩하고 싶다면,
Homebrew 를 사용해 인스톨한다. 오리지널 ctags 는 /usr/bin/ctags 에 있을 것이고,
새로 인스톨된 것은 /usr/local 아래 어딘가 있을 듯.
     
    $ brew install ctags-exuberant

/usr/local/bin/ctags 가 실행되도록 확인하고, 마찬가지로 ~/.ctags 에 

    --python-kinds=-i

추가.

ctags 에 관한 좋은 블로그 [Code Spelunking with Ctags and Vim](http://www.scholarslab.org/research-and-development/code-spelunking-with-ctags-and-vim/)

그리고 pycscope 을 쓸수도 있는데,

OS X 에서는 `brew install cscope` 해주고, cscope_maps.vim 을 ~/vim/plugin 아래에 설치한다.
그리고 pycscope 도 설치한다. `pycscope -R .` cscope_maps.vim 도 추가.

[The Vim/Cscope tutorial](http://cscope.sourceforge.net/cscope_vim_tutorial.html)

개인적으로는 cscope 이 ctags 보다 편하다. 

`Ctrl-\`+`c` 나 `Ctrl-\`+`s` 그리고, `Ctrl-\`+`d`

### ack & z jumper

ack 가 grep 보다 소스코드에서 뭐 찾을때 훨 편하다.

그리고 디렉토리 navigation 은 z jumper 가 최고다. 예전의 노턴 ncd 같은 녀석.

### bash

스크린캐스트에서 본 쉘 커맨드.

    ~/code/jade[master]% find . -iname "*.rb" | 
    xargs grep -h '^[[:space:]]*class\|module\b' | 
    sed 's/^[[:space:]]*//' | grep -v '^#' | cut -d ' ' -f 2 | 
    while read class; 
    do echo `grep -rl "\b$class\b" app lib --include "*.rb" |wc -l` $class ; 
    done | sort -n

한 줄이다. ruby 의 클래스를 참조하는 화일 갯수를 세는건데.

* grep -l 옵션은 화일에서 패턴 존재가 확인되면 그냥 스킵. 존재 여부를 확인할때.
* grep -r 은 디렉토리 리커시브.
* grep -v 는 패턴이 없는 라인을 출력.
* grep -h 는 화일 이름을 출력 안함.
* find ... | xargs 가 find ... -exec 보다 빠르다.
* sort -n 은 numerical sort 를 하란 것인데, 그냥 sort 하면 "10" 이 "2" 보다 앞에 나온다. -n 을 해주면 "2" 가 "10" 보다 먼저 나오게 된다.

### zsh

zsh 에서는 Global alias 이용해

    alias -g G='|grep '

하면 

<pre>
cat log G ERROR
</pre>

와 같은 커맨드가 가능하다.
### bash I/O redirection

[Bash While Loop Example](http://www.cyberciti.biz/faq/bash-while-loop/) 에서.

    cmd >>file.txt 2>&1

 stderr, stdout 을 함께 화일에 append 할 때.

    #!/bin/bash
    FILE=$1
    exec 3<&0
    exec 0<$FILE
    while read line
    do
      echo $line
    done
    exec 0<&3

 stdin 을 3 에 카피해놓고, 화일을 stdin 으로 돌리고, 나중에 원복.

    #!/bin/bash
    x=1
    while [ $x -le 5 ]
    do
      echo "$x times"
      x=$(( $x + 1 ))
    done

bash 의 arithmetic operator 는 (( )) 이다.

    #!/bin/sh
    counter=$1
    factorial=1
    while [ $counter -gt 0 ]
    do
      factorial=$(( $factorial * $counter ))
      counter=$(( $counter - 1 ))
    done
    echo $factorial

factorial 구하는 것.

    #!/bin/sh
    while :
    do
      read -p "Enter two numbers ( -1 to quit ) : " a b
      if [ $a -eq -1 ]
      then
        break
      fi
      ans=$(( a + b ))
      echo $and
    done

읽을만한 shell scripting 문서: [Linux Shell Scripting Tutorial (LSST) v2.0](http://bash.cyberciti.biz/guide/Main_Page)
