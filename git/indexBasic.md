# Базовый Git

## Файл md

Выделение:
*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

_You **can** combine them_

Картинки:
![GitHub Logo](/images/logo.png)
Format: ![Alt Text](url)



Задачи:
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item

Таблицы:
First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column


## Настройки файла .gitconfig

```csharp
[alias]
	ch = checkout
	com = commit
	st = status
	br = branch
	hist = log --pretty=format:'%C(yellow)%h %C(bold cyan)%ad(%ar) %C(reset)| %s%C(red)%d %C(dim white)[%an]' --graph --date=short
	lg = log --oneline --abbrev-commit --graph
	last = log -1 HEAD --stat
	unstage = reset HEAD --
```
