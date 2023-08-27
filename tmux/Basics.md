
tmux is a terminal multiplexer. It lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.

## Leader key

tmux use "Ctrl+b" as its leader key.

Leader key is the key (or key combination), with which all the key combination start with.

## Split window

1) _Leader_ + % 
	Split pane vertically
2) _Leader_ + "
	Split pane horizontally

## Motions

_Leader_ + Arrow keys (Left, Right, Up, Down)

## Change pane layout

_Leader_ + Space

## Create new pane

_Leader_ + c

## Selecting specific pane

_Leader_ + "Pane number"

	Examples - 
	- Leader + 1
	- Leader + 0

## Renaming pane

1) _Leader_ + :
2) rename-window + Enter
3) type name + Enter


## Detach from session / Put tmux in background

_Leader_ + d


## Attach to tmux session

tmux attach-session

## List tmux sessions

tmux ls

## Create new session

tmux

## List sessions inside tmux

_Leader_ + s

We can then use arrow key to switch sessions

## Change session name

1) Leader + :
2) rename-session + enter
3) session name + enter


## List window in sessions

_Leader_ + w


