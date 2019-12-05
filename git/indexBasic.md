# Базовый Git





Настройки файла .gitconfig

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