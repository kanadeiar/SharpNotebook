# Примеры Android

Скрытие клавиатуры:
```csharp
var imm = (InputMethodManager)GetSystemService(Context.InputMethodService);
var focus = this.CurrentFocus;
if (focus != null)
    imm.HideSoftInputFromWindow(focus.WindowToken, HideSoftInputFlags.None);
```

