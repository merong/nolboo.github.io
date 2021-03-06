---
layout: post
title: "Vim에서 저장하는 방법 - 자동 저장"
description: "Vim에서 저장하는 가장 기초적인 명령과 자동 저장을 할 수 있는 플러그인을 소개한다"
category: blog
tags: [vi, vim, write, save, tip]
---

![Vim 3D](/images/posts/vim.jpg)

Vim은 25년 된 텍스트 에디터이고, 조상인 vi와 호환성을 남겨두기 위해 (어찌 보면 갸륵하기까지 한) 노력을 하는 에디터라고 생각한다.

파일을 메모리로 불러들여 편집한 후 저장하는 프로세스는 (버퍼라는 용어만 빼면) 최근에 나오는 에디터와 다른 바가 없다. 그러나, vi와 이전 버전과의 호환성을 이유로(?) 수동으로 저장을 하는 것이 기본으로 하고 있다.

메모리에서 편집한 버퍼를 저장하는 명령 중 가장 많이 알려진 ":write"는 ":w"로 줄여 쓸 수 있다.

```vim
:w<CR>
"filename.txt" 42L, 1200C written
```

* 명령 모드에서 모든 명령은 <CR>로 끝낸다.
* 저장한 결과로 "파일명"과 파일의 크기, 저장함(written)으로 메시지를 보여준다.

그러나, 저장할 때마다 세 번의 키 입력을 한다는 것은 아무래도 불편하다. 그래서 많은 사용자가 키매핑을 `~/.vimrc` 파일에 추가한다.

```vim
nmap <C-S> :w<CR>
imap <C-S> <ESC>:w<CR>a
```

`<CTRL-S>` 키를 ":w<CR>"로 매핑하고, 입력 모드에서는 일반 모드로 나온 후(`<Esc>`) ":w<CR>"로 저장한 후 다시 "a" 명령으로 커서 위치로 돌아오게 된다.

입력 모드에서 일반 모드 명령을 하나 실행할 수 있는 `<CTRL-O>` 명령을 이용하면 Vim이 알아서 커서 위치를 지켜준다.

```vim
imap <C-S> <C-O>:w<CR>
```

그러나, 이 방법은 `<CTRL-S>`가 제대로 매핑되지 않는 시스템이 있다. `<CTRL-S>`가 아닌 다른 키나 기능키를 매핑할 수도 있지만, 입력 모드에서 기능키가 그냥 텍스트로 입력되는 시스템도 있다.

그래서인지 일반 모드에서 리더키를 이용하는 방법도 많이 사용하고 있다.

```vim
nnoremap <Leader>w :w<CR>
```

":w" 명령과도 비슷하고 기억하기도 쉬워서 많은 사용자가 애용하는 방법이다.

그러나, 최근 텍스트 에디터는 대부분 텍스트가 입력되지 않는 시간을 이용한 자동 저장 기능을 기본으로 제공하니 아직도 만족스럽지 않다. 그래서 방법을 찾던 중 발견한 것이 [AutoSave](https://github.com/907th/vim-auto-save) 플러그인이다.

AutoSave 플러그인인 일반 모드에서 변경하거나 입력 모드에서 벗어날 때마다 ":w" 명령을 실행하여 파일을 저장한다.

Vim 플러그인 관리자를 사용하고 있다면 vimrc 파일에서 쉽게 추가할 수 있다. [vim-plug](https://github.com/junegunn/vim-plug)를 사용하면:

```vim
Plug '907th/vim-auto-save'
let g:auto_save = 1
let g:auto_save_silent = 1
```

"let g:auto_save = 1"은 Vim이 시작할 때 자동 저장을 기본으로 한다. "let g:auto_save_silent = 1"은 저장할 때 나오는 메시지를 보여주지 않는다.

2분간 텍스트를 입력하지 않으면 자동 저장하는 다른 텍스트 에디터와는 약간 다르지만, Vim에서의 변경은 언제든지 반복할 수 있어야 하고, 실행 취소의 단위도 되기 때문에 이 방법이 합리적이라고 생각된다.

몇 개월간 사용해 보니 안정적이고, 다른 방법보다 편리하다. 다만 지금처럼 블로그 글을 쓸 때 지킬을 로컬로 실행하고 있으면 저장될 때마다 제너레이트하기 때문에 미리보기 지연이 많다. 이럴 때는 AutoSave에서 제공하는 "AutoSaveToggle" 명령으로 자동 저장을 끌 수 있다.

```vim
:AutoSaveToggle<CR>
```

같은 명령으로 다시 켤 수 있다. 토글 명령이다.

즐빔!

## 자동 저장 이벤트

AutoSave를 사용할 때 일반 모드 변경이 일어날 때마다 저장하면 매크로를 실행할 때 느려지는 것이 싫을 때가 있다는 제보가 있다. 이럴 때는 [저장 이벤트](https://github.com/907th/vim-auto-save#events)를 지정할 수 있다.

g:auto_save_events 옵션은 앞서 말한 바와 같이 일반 모드 변경(TextChanged)과 입력 모드를 벗어날 때(InsertLeave)가 기본값으로 되어 있다. vimrc 파일에 다음과 같이 입력한 것과 같다.

```vim
let g:auto_save_events = ["InsertLeave", "TextChanged"]
```

일반 모드 변경을 빼고 싶다면 다음과 같이 TextChanged를 빼면 된다.

```vim
let g:auto_save_events = ["InsertLeave"]
```

Vim이 인식할 수 있는 다른 이벤트들도 추가할 수 있다(:h autocmd-events). AutoSave가 추천하는 이벤트는 입력 모드 변경(TextChangedI), 키를 일정 시간 입력하지 않음(CursorHold, CursorHoldI), 입력 모드 자동 완성(CompleteDone)이 있다. 다만, CursorHold, CursorHoldI는 시간을 1000 이상 설정을 권한다.

## Esc 키매핑

`<Esc>`를 누를 때마다 저장하도록 키매핑하려면 다음을 vimrc 파일에 추가한다.

```vim
inoremap <silent> <Esc> <Esc>:w<CR>
```

`<Esc>`키를 누를 때마다 저장된다. `<silent>`는 ":w<CR>" 명령을 보이지 않게 한다. 그러나, 파일 저장 메시지는 보인다. 앞서 말한 바와 같이 `<CTRL-S>`와 `<Esc>` 키매핑 방법은 (맥 터미널과 같은) 특정 환경 에서는 제대로 동작하지 않을 때가 있다.

>AutoSave에서는 'auto_save_silent' 옵션으로 저장 메시지도 보이지 않게 할 수 있다.

## autowriteall 옵션

사실 Vim 자체에서도 자동 저장 옵션 'autowrite'와 'autowriteall'을 기본 제공하고 있다. ":next", ":make", `<CTRL-O>`와 `<CTRL-I>` 점프 명령과 ":edit", ":quit" 명령을 내릴 때 파일의 변경이 있으면 저장한다.

'autowrite'보다 'autowriteall'이 더 많은 명령을 - 특히 ":e", ":q" - 지원한다. vimrc 파일에 다음을 추가한다.

```vim
set autowriteall
```

끝!
