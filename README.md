
# fd
[![Build Status](https://travis-ci.org/sharkdp/fd.svg?branch=master)](https://travis-ci.org/sharkdp/fd)
[![Build status](https://ci.appveyor.com/api/projects/status/21c4p5fwggc5gy3j/branch/master?svg=true)](https://ci.appveyor.com/project/sharkdp/fd/branch/master)
[![Version info](https://img.shields.io/crates/v/fd-find.svg)](https://crates.io/crates/fd-find)
[中文](https://github.com/chinanf-boy/fd-zh)

*fd*는 *[find](https://www.gnu.org/software/findutils/)* 를 대체할 수 있는 간단하고, 빠르고, 그리고 사용자 친화적인 commad line util이다.

이 fd는 *find*의 강력한 모든 기능을 지원하지 않지만, 주요 기능의 [80%](https://en.wikipedia.org/wiki/Pareto_principle)정도 지원하고 있다.

## 특징
- 편리한 문법(syntax): `find -iname '*PATTERN*'`대신에 `fd PATTERN`.
- Terminal에서 컬러풀한 결과(*ls*와 유사하게).
- 빠른 속도 (참고: [benchmarks](#benchmark)).
- Smart case: 기본적으로 case-insensitive한 검색을 지원한다. 패턴에 대문자가 포함되어 있으면 case-sensitive하게 변경된다[\*](http://vimdoc.sourceforge.net/htmldoc/options.html#'smartcase').
- 숨김 폴더 및 파일은 무시(default).
- `.gitignore`에서 설정한 내용은 무시(default).
- 정규식 지원.
- 유니코드(unicode) 지원.
- Command가 `find`보다 50%정도 짧음 :-).
- GNU 병렬과 유사한 문법(syntax)으로 병렬 실행 command 지원.

## 데모
![Demo](doc/screencast.svg)

## 벤치마크
벤치마크를 하기 위해 home 폴더 안에 있는 `[0-9].jpg`로 끝나는 파일을 검색할 것이다. Home 폴더에는 약 190,000개 이상 되는 하위 폴더들과 약 백 만개의 파일들이 있다. 또 벤치마크 결과를 분석(averaging and statistical analysis)하기 위해 [hyperfine](https://github.com/sharkdp/hyperfine)을 사용한다. 
아래의 벤치마크 결과는 "warm"/pre-filled disck-cache 환경에서 수행한 결과다("cold" disk-cache 환경에서도 유사한 트랜드가 나타남).

`find`를 수행했을 때:
```
    Benchmark #1: find ~ -iregex '.*[0-9]\\.jpg$'
    
      Time (mean ± σ):      7.236 s ±  0.090 s
    
      Range (min … max):    7.133 s …  7.385 s
```

`find`는 정규식을 사용하지 않는 검색에서 꽤 빠른 결과를 보여주고 있다: 
```
    Benchmark #2: find ~ -iname '*[0-9].jpg'
    
      Time (mean ± σ):      3.914 s ±  0.027 s
    
      Range (min … max):    3.876 s …  3.964 s
```

이제 `fd`를 이용해서 같은 작업을 해본다. 주의할 점은 `fd` 는 항상 정규식으로 검색한다. 옵션으로 넣은 `--hidden`과 `--no-ignore`은 위와 동일한 비교를 위해 필요했다. 왜냐하면 `fd`는 숨김 폴더들과 ignored 된 경로들을 기본적으로 검색하지 않기 때문이다(아래를 참고):
```
    Benchmark #3: fd -HI '.*[0-9]\\.jpg$' ~
    
      Time (mean ± σ):     811.6 ms ±  26.9 ms
    
      Range (min … max):   786.0 ms … 870.7 ms
```

위의 상황에서, `fd`가 대략적으로 `find -iregex`보다 9배, `find -iname`보다 5배나 더 빠른 성능을 보여주고 있다. 성능 차이가 나는 반면에 두개의 결과 모든 같은 결과를 보여주었다 :smile:.

끝으로, `--hidden`과 `--no-ignore` 옵션 없이 실행 해보자(물론 이 검색 결과는 위와 다르다). *fd*로 숨겨진 폴더 또는 git-ignored로 무시되는 것을 찾을 필요가 없다면, 훨씬 더 빠르다:
```
    Benchmark #4: fd '[0-9]\\.jpg$' ~
    
      Time (mean ± σ):     123.7 ms ±   6.0 ms
    
      Range (min … max):   118.8 ms … 140.0 ms
```

**Note**: 이 벤치마크는 *하나의 특정 machine*에서 진행한 결과이다. 꽤 많은 테스트(아래에 계속되는 테스트 결과)를 진행했지만, 이 결과는 다른 machine에서의 결과와 다를 수 있다! 다른 사람들이 이 테스트를 자신의 컴퓨터에서 테스트를 해보는 것을 기대한다. 필요한 모든 스크립트는 이 [this repository](https://github.com/sharkdp/fd-benchmarks)를 참고하자.

Concerning *fd*'s speed, the main credit goes to the `regex` and `ignore` crates that are also used
in [ripgrep](https://github.com/BurntSushi/ripgrep) (check it out!).

## 컬러풀한 결과
`fd`는 `ls`처럼 extension으로 결과를 컬러풀하게 나타낼 수 있다. 컬러풀하게 표현하기 위해서는 [`LS_COLORS`](https://linux.die.net/man/5/dir_colors)라는 환경 변수가 반드시 설정되어 있어야 한다. 일반적으로 이 값은 `dircolors`에 의해 설정된다. `dircolors`라는 command는 다른 형식의 파일을 색깔로 구분하기 위해 편리한 설정 포맷을 제공하고 있다. 보편적으로 `LS_COLORS`는 이미 설정되어 있다. 만약 다른 색상 또는 더 이쁜 색상을 설정하고 싶다면, [here](https://github.com/seebi/dircolors-solarized) 또는 [here](https://github.com/trapd00r/LS_COLORS)를 참고하자.  

`fd`는 [`NO_COLOR`](https://no-color.org/) 환경 변수도 존중한다.

## 병렬 Command 실행
Command에 `-x`/ `--exec`라는 옵션이 있다면, 병렬로 command를 수행하기 위한 job pool이 생성될 것이다. 문법은 GNU 병렬 처리와 유사하다:

- `{}`: 검색 결과의 경로로 대체되는 플레이스 홀더(`documents/images/party.jpg`).
- `{.}`: `{}`와 비슷하나 파일의 확장자가 없는 플레이스 홀더(`documents/images/party`).
- `{/}`: 검색 결과의 basename로 대체되는 플레이스 홀더(`party.jpg`).
- `{//}`: 찾은 경로의 상위(부모) 경로를 이용하는 플레이스 홀더(`documents/images`).
- `{/.}`: 파일 확장자가 제거된 basename을 사용하는 플레이스 홀더(`party`).

``` bash
# Convert all jpg files to png files:
fd -e jpg -x convert {} {.}.png

# Unpack all zip files (if no placeholder is given, the path is appended):
fd -e zip -x unzip

# Convert all flac files into opus files:
fd -e flac -x ffmpeg -i {} -c:a libopus {.}.opus

# Count the number of lines in Rust files (the command template can be terminated with ';'):
fd -x wc -l \; -e rs
```

Command를 실행하기 위한 쓰레드(thread)의 수는 `--threads`/ `-j`옵션으로 설정할 수 있다.

## 설치

### On Ubuntu
*... and other Debian-based Linux distributions.*

If you run Ubuntu 19.04 (Disco Dingo) or newer, you can install the
[officially maintained package](https://packages.ubuntu.com/disco/fd-find):
```
sudo apt install fd-find
```
Note that the binary is called `fdfind` as the binary name `fd` is already used by another package.
It is recommended that you add an `alias fd=fdfind` to your shells initialization file, in order to
use `fd` in the same way as in this documentation.

If you use an older version of Ubuntu, you can download the latest `.deb` package from the
[release page](https://github.com/sharkdp/fd/releases) and install it via:
``` bash
sudo dpkg -i fd_7.5.0_amd64.deb  # adapt version number and architecture
```

### On Debian

If you run Debian Buster or newer, you can install the
[officially maintained Debian package](https://tracker.debian.org/pkg/rust-fd-find):
```
sudo apt-get install fd-find
```
Note that the binary is called `fdfind` as the binary name `fd` is already used by another package.
It is recommended that you add an `alias fd=fdfind` to your shells initialization file, in order to
use `fd` in the same way as in this documentation.

### On Fedora

Starting with Fedora 28, you can install `fd` from the official package sources:
``` bash
dnf install fd-find
```

For older versions, you can use this [Fedora copr](https://copr.fedorainfracloud.org/coprs/keefle/fd/) to install `fd`:
``` bash
dnf copr enable keefle/fd
dnf install fd
```

### On Alpine Linux

You can install [the fd package](https://pkgs.alpinelinux.org/packages?name=fd)
from the official sources, provided you have the appropriate repository enabled:
```
apk add fd
```

### On Arch Linux

You can install [the fd package](https://www.archlinux.org/packages/community/x86_64/fd/) from the official repos:
```
pacman -S fd
```
### On Gentoo Linux

You can use [the fd ebuild](https://packages.gentoo.org/packages/sys-apps/fd) from the official repo:
```
emerge -av fd
```

### On openSUSE Linux

You can install [the fd package](https://software.opensuse.org/package/fd) from the official repo:
```
zypper in fd
```

### On Void Linux

You can install `fd` via xbps-install:
```
xbps-install -S fd
```

### On macOS

You can install `fd` with [Homebrew](https://formulae.brew.sh/formula/fd):
```
brew install fd
```

… or with MacPorts:
```
sudo port install fd
```

### On Windows

You can download pre-built binaries from the [release page](https://github.com/sharkdp/fd/releases).

Alternatively, you can install `fd` via [Scoop](http://scoop.sh):
```
scoop install fd
```

Or via [Chocolatey](https://chocolatey.org):
```
choco install fd
```

### On NixOS / via Nix

You can use the [Nix package manager](https://nixos.org/nix/) to install `fd`:
```
nix-env -i fd
```

### On FreeBSD

You can install [the fd-find package](https://www.freshports.org/sysutils/fd) from the official repo:
```
pkg install fd-find
```

### From NPM

On linux and macOS, you can install the [fd-find](https://npm.im/fd-find) package:

```
npm install -g fd-find
```

### From source

With Rust's package manager [cargo](https://github.com/rust-lang/cargo), you can install *fd* via:
```
cargo install fd-find
```
Note that rust version *1.36.0* or later is required.

### From binaries

The [release page](https://github.com/sharkdp/fd/releases) includes precompiled binaries for Linux, macOS and Windows.

## Development
```bash
git clone https://github.com/sharkdp/fd

# Build
cd fd
cargo build

# Run unit tests and integration tests
cargo test

# Install
cargo install
```

## Command-line options
```
USAGE:
    fd [FLAGS/OPTIONS] [<pattern>] [<path>...]

FLAGS:
    -H, --hidden            Search hidden files and directories
    -I, --no-ignore         Do not respect .(git|fd)ignore files
        --no-ignore-vcs     Do not respect .gitignore files
    -s, --case-sensitive    Case-sensitive search (default: smart case)
    -i, --ignore-case       Case-insensitive search (default: smart case)
    -g, --glob              Glob-based search (default: regular expression)
    -F, --fixed-strings     Treat the pattern as a literal string
    -a, --absolute-path     Show absolute instead of relative paths
    -L, --follow            Follow symbolic links
    -p, --full-path         Search full path (default: file-/dirname only)
    -0, --print0            Separate results by the null character
    -h, --help              Prints help information
    -V, --version           Prints version information

OPTIONS:
    -d, --max-depth <depth>            Set maximum search depth (default: none)
    -t, --type <filetype>...           Filter by type: file (f), directory (d), symlink (l),
                                       executable (x), empty (e)
    -e, --extension <ext>...           Filter by file extension
    -x, --exec <cmd>                   Execute a command for each search result
    -X, --exec-batch <cmd>             Execute a command with all search results at once
    -E, --exclude <pattern>...         Exclude entries that match the given glob pattern
    -c, --color <when>                 When to use colors: never, *auto*, always
    -S, --size <size>...               Limit results based on the size of files.
        --changed-within <date|dur>    Filter by file modification time (newer than)
        --changed-before <date|dur>    Filter by file modification time (older than)

ARGS:
    <pattern>    the search pattern - a regular expression unless '--glob' is used (optional)
    <path>...    the root directory for the filesystem search (optional)
```

This is the output of `fd -h`. To see the full set of command-line options, use `fd --help` which
also includes a much more detailed help text.

## 튜토리얼

우선, command line의 모든 옵션들을 개괄적으로 알아보기 위해, 두 개의 command를 사용할 수 있다. 간략하게 메시지를 보기 위해서는 `fd -h`, 더 상세한 메세지를 보기 위해서는 `fd --help`.

### 간단하게 검색하기

*fd*는 파일시스템(filesystem)의 모든 것을 찾기 위해 만들어졌다. 가장 기본적인 검색 방법은 검색 패턴(search pattern)을 이용하여 *fd*를 실행하는 것이다. 예를 들어, `netflix`라는 이름이 포함된 오래된 script를 찾는다고 하자:
``` bash
> fd netfl
Software/python/imdb-ratings/netflix-details.py
```
위와 같이 검색 패턴을 주어 command를 실행하면, *fd*는 현재 폴더부터 재귀적으로 `netfl`라는 패턴이 포함된 모든 것들을 검색한다.

### 정규 표현식으로 검색하기

검색 패턴은 정규 표현식으로 처리한다. 여기서 우리는 `x`로 시작하고 `rc`로 끝나는 모든 것을 검색해보자:
``` bash
> cd /etc
> fd '^x.*rc$'
X11/xinit/xinitrc
X11/xinit/xserverrc
```

### 상위 폴더 지정해서 검색하기
특정 폴더에서 검색하고 싶다면, *fd*에 두 번째 매개변수(argument)로 넘겨줄 수 있다:
``` bash
> fd passwd /etc
/etc/default/passwd
/etc/pam.d/passwd
/etc/passwd
```

### 어떠한 매개변수 없이 *fd* 실행하기

*fd*는 어떠한 매개변수 없이 실행할 수 있다. 이 방법은 재귀적으로(`ls -R`과 유사하게) 아주 빠르게 현재 폴더의 모든 것을 보는데에 아주 유용하다:
``` bash
> cd fd/tests
> fd
testenv
testenv/mod.rs
tests.rs
```

주어진 폴더 밑의 모든 파일들을 보는 기능을 사용하고 싶다면, `.`와 `^`같은 catch-all 패턴을 이용해야 한다:
``` bash
> fd . fd/tests/
testenv
testenv/mod.rs
tests.rs
``` 

### 특정 확장자로 검색하기

때때로, 특정 확장자의 모든 파일들을 찾고 싶을 수 있다. 이때는 -e(or `--extension`)라는 옵션을 이용할 수 있다. fd 저장소(repository)에 있는 곳에서 마크다운 확장자의 모든 파일을 찾아 보자:
``` bash
> cd fd
> fd -e md
CONTRIBUTING.md
README.md
``` 

이 `-e`라는 옵션은 검색 패턴과 조합하여 사용할 수 있다:
``` bash
> fd -e rs mod
src/fshelper/mod.rs
src/lscolors/mod.rs
tests/testenv/mod.rs
```

### 숨김, 무시되는 파일들

기본적으로 fd는 숨김 폴더, 파일들을 검색하지 않는다. 따라서 숨겨진 파일 또는 폴더를 찾기 위해서, `-H` (or `--hideen`) 옵션을 이용한다:
``` bash
> fd pre-commit
> fd -H pre-commit
.git/hooks/pre-commit.sample
```

Git 저장소(또는 Git 저장소들을 포함하는 폴더)에서 작업을 한다면, fd는 `.gitignore`의 패턴에 매칭되는 것을 검색하지 않는다. 위와 마찬가지로 이 기능을 비활성하기 위해, `-I` (or `--no-ignore`) 옵션을 이용한다:
``` bash
> fd num_cpu
> fd -I num_cpu
target/debug/deps/libnum_cpus-f5ce7ef99006aa05.rlib
```

즉, 모든 파일, 폴더들을 검색하기 위해서는 간단하게 위의 두 옵션을 조합하여 검색한다(`-HI`).

### 특정 폴더 및 파일들을 제외하기

가끔 우리는 특정 하위 폴더의 검색 결과를 제외하고 싶을 수 있다.  
예를 들어, `.git`에 있는 모든 것을 제외하고 `-H` 옵션을 이용해 모든 숨김 파일, 폴더들을 찾는다고 하자. 이때 `-E` (or `--exclude`) 옵션을 사용할 수 있는데, 이 옵션은 arbitary glob 패턴으로 매개변수를 입력 받는다:
``` bash
> fd -H -E .git …
```

또한 이렇게 마운트 된 폴더들을 제외할 수 있다:
``` bash
> fd -E /mnt/external-drive …
```

.. 또는 특정 파일 형식을 제외할 수 있다:
``` bash
> fd -E '*.bak' …
```

이러한 제외 패턴들을 지속적으로 저장할 수 있는데, 이때 `.fdignore` 파일을 이용할 수 있다. 이 파일은 `.gitignore`파일과 비슷하게 동작하는데, `fd`에 한정되어 있다. 예를 들어:
``` bash
> cat ~/.fdignore
/mnt/external-drive
*.bak
```
Note: 또한 `fd`는 `rg`나 `ag`와 같은 다른 프로그램에서 사용하는 `.ignore` 파일을 지원한다.

### `xargs` 또는 `parallel`와 함께 사용하기

모든 검색 결과들을 활용하고 싶다면, `xargs`에 결과를 전달(pipe)할 수 있다:
``` bash
> fd -0 -e rs | xargs -0 wc -l
```

여기서 `-0` 옵션은 fd에게 NULL(개행 대신에)문자로 검색 결과를 구분하도록 한다. 이와 같은 방식으로 `xargs`의 `-0` 옵션도 NULL 문자로 구분하여 입력을 받도록 한다.

### 파일들을 삭제하기

검색 패턴에 충족하는 모든 파일 또는 폴더를 삭제하는데, `fd`를 이용할 수 있다. 만약 파일들만 삭제하기를 원한다면, `--exec-batch`/ `-X` 옵션을 이용하여 `rm`을 실행할 수 있다. 예를 들어, 재귀적으로 모든 `.DS_Store` 파일을 삭제하기 위해서는 다음과 같이 실행한다:
``` bash
> fd -H '^\.DS_Store$' -tf -X rm
```
삭제하기 전에 결과를 보고 싶다면, `-X rm` 옵션을 주지 않고 `fd`를 실행해보자. 아니면 다른 방법으로 "interactive"라는 `rm`의 옵션을 이용하자:
``` bash
> fd -H '^\.DS_Store$' -tf -X rm -i
```

또, 폴더를 삭제하고 싶다면, 같은 방법으로 제거할 수 있다. `rm`의 `--recursive`/ `-r` 옵션을 이용해 폴더를 삭제할 수 있다.

Note: `fd … -X rm -r`와 같은 명령어를 실행할 때, race conditions을 초래할 수 있다: 예를 들자면, `…/foo/bar/foo/…`와 같이 폴더가 존재하고, `foo`라는 모든 폴더를 삭제한다고 하자. 이때 먼저 부모 경로의 `foo`폴더가 삭제되고 난 후, `rm`을 수행한다면 *"'foo/bar/foo':
No such file or directory"* 라는 에러가 나타는 상황을 만나게 될 것이다.

### Troubleshooting

### `fd`로 파일이 정상적으로 찾을 수 없다면?!

기본적으로 `fd`는 숨김 파일, 폴더들은 검색하지 않는 것을 기억하자. 또 `.gitignore`의 패턴에 만족하는 것들을 무시한다. 반드시 가능한 모든 파일을 찾고 싶다면, `-H`와 `-I`라는 옵션을 이용해 두 기능을 비활성화 하자.
``` bash
> fd -HI …
```

### `fd`에서 정규식 패턴이 정상적으로 동작하지 않는다면?!

많은 특수 문자(`[]`, `^`, `$`, ... 처럼)는 쉘(shell)에서도 특수 문자들이다. 의심스럽다면, 정규식 패턴을 single quote(')로 감싸서 사용하자: 
``` bash
> fd '^[A-Z][0-9]+$'
```

만약 dash(-)로 시작하는 패턴을 사용한다면, 반드시 command 끝에 `--`을 옵션에 추가해야한다. 그러면 command의 옵션으로 그 패턴이 정상 동작할 것이다. 또 다른 방법으로는 hyphen(-)을 character class로 사용하자: 

``` bash
> fd -- '-pattern'
> fd '[-]pattern'
```

### 다른 프로그램들과 통합하기

#### `fzf`와 함께 fd 사용하기

fuzz finder [fzf](https://github.com/junegunn/fzf)의 입력을 *fd*를 이용해 생성할 수 있다:
``` bash
export FZF_DEFAULT_COMMAND='fd --type file'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
```

위와 같이 설정한 후에 fd 결과를 검색하고 fzf를 사용하기 위해 터미널에서 `vim <Ctrl-T>`를 입력할 수 있다..

또 숨김 파일들과 심볼릭 링크 파일들을 포함하고 싶을 수 있다:
``` bash
export FZF_DEFAULT_COMMAND='fd --type file --follow --hidden --exclude .git'
```    

설정에 의해 fzf 안에서도 컬러풀하게 fd의 결과를 볼 수 있다.
``` bash
export FZF_DEFAULT_COMMAND="fd --type file --color=always"
export FZF_DEFAULT_OPTS="--ansi"
```    

더 상세한 내용은 fzf REAME에 있는 [Tips section](https://github.com/junegunn/fzf#tips)을 참고하자

### `emacs`와 함께 fd 사용하기

emacs의 패키지인 [find-file-in-project](https://github.com/technomancy/find-file-in-project)도 파일들을 찾기 위해 *fd*를 사용할 수 있다.

`find-file-in-project`을 설치한 후에, `(setq ffip-use-rust-fd t)`을 `~/.emacs` 또는 `~/.emacs.d/init.el` 파일에 추가하자.

emacs에서는 파일들을 찾기 위해, `M-x find-file-in-project-by-selected`을 실행 해보자. 또, 프로젝트의 모든 파일을 보기 위해서는 `M-x find-file-in-project`을 실행 해보자.

## License

Copyright (c) 2017-2020 The fd developers

`fd` is distributed under the terms of both the MIT License and the Apache License 2.0.

See the [LICENSE-APACHE](notion://www.notion.so/spearkkk/LICENSE-APACHE) and [LICENSE-MIT](notion://www.notion.so/spearkkk/LICENSE-MIT) files for license details.
