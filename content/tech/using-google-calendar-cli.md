---
title: Using Google Calendar CLI
date: 2026-10-03T20:03:00.000-07:00
author: Sitesh Pattanaik
---
I was exploring a way to use google calendar via my terminal and without building any OAuth system but still keeping it public.

Came across this cool library about <https://github.com/insanum/gcalcli> and I gotta say, its pretty neat. Though the documentation and usages could have been a bit better/easier, but I guess claude can parse through and help me with commands that I need for my use case without much efforts.

So, using claude, I build my own shorthand wrapper that i can use on a daily basis for managing my calendar via CLI.

Here is the configuration that makes sense to me and hope it helps you make your life easier too. Thanks to all who have helped built `gcalcli`

```
# ~/.zshrc

unalias gcal 2>/dev/null
gcal() {
  local cal="<email address of the google account>"
  case "$1" in
    w)      gcalcli --calendar "$cal" calw today 1 ;;
    w2)     gcalcli --calendar "$cal" calw today 2 ;;
    a)      gcalcli --calendar "$cal" agenda today +14d ;;
    q)      shift; gcalcli --calendar "$cal" quick "$@" ;;
    e)      shift; gcalcli --calendar "$cal" edit "$@" ;;
    d|del)  shift; gcalcli --calendar "$cal" delete "$@" ;;
    *)      gcalcli --calendar "$cal" "$@" ;;
  esac
}
```
