# Компоненты Xamarin.Forms

## Отступы

Margin, Padding

```xml
<StackLayout Padding="0,20,0,0">
    <BoxView Color="Green" Margin="20" />
    <BoxView Color="Blue" Margin="10, 15" />
    <BoxView Color="Red" Margin="0, 20, 15, 5" />
</StackLayout>
```

## Выравнивание горизонтальное и вертикальное

Start, Center, End, Fill, StartAndExpand, CenterAndExpand, EndAndExpand, FillAndExpand

```xml
<StackLayout Padding="0,20,0,0">
    <Label Text="Привет из Xamarin Forms" VerticalOptions="Center" HorizontalOptions="Center" />
</StackLayout>
```

## Выравнивание текста

Start, Center, End

```xml
<StackLayout Padding="0,20,0,0">
    <Label Text="Привет из Xamarin Forms" VerticalTextAlignment="Center" HorizontalTextAlignment="Center" />
</StackLayout>
```

## Кнопки

xaml
```xml
<Button Text = "Нажми!" FontSize="Large" BorderWidth="1"
    HorizontalOptions="Center" VerticalOptions="CenterAndExpand" Clicked="OnButtonClicked" />
```
Код
```csharp
private void OnButtonClicked(object sender, System.EventArgs e)
{
    Button button = (Button)sender;
    button.Text = "Нажато!";
    button.BackgroundColor = Color.Red;
}
```

## Текстовые поля

- Label: текстовая метка для вывода текста

- Entry: однострочное текстовое поле

- Editor: многострочное текстовое поле

Основные свойства элемента Entry, которые мы можем использовать:

Text: введенный в поле текст

TextColor: цвет вводимого текста

Placeholder: текст-заменитель, который отображается в поле до ввода и исчезает при начале печати

IsPassword: при значении true указывает, то поле будет предназначено для ввода пароля, и поэтому все вводимые символы будут заменяться звездочкой

Также класс Entry определяет два события:

TextChanged: возникает при вводе символов в поле

Completed: возникает при завершении ввода

Среди свойств текстового поля ввода наиболее интересным является свойство Keyboard.

- Default

- Text

- Chat

- Url

- Email

- Telephone

- Numeric

xaml
```xml
<StackLayout>
    <Label Font="26" Text="Многострочный ввод" />
    <Editor BackgroundColor="#a5d6a7" HeightRequest="200" />
</StackLayout>
```

## Контейнер выделения

xaml
```xml
<Frame BorderColor="Gray" BackgroundColor="#e1e1e1" CornerRadius="8">
    <Label Text="Xamarin Forms" FontSize="Large" HorizontalOptions="Center"  />
</Frame>
```

## Изображения

Добавление для Android. В проекте для Android добавить в папку Resources/Drawable. А в окне Properties установим у этого изображения Build Action: AndroidResource.

Добавление для iOS. В проекте для iOS перейдем к папке Assets Catalogs. По умолчанию в ней должен быть каталог Assets. В открывшмся окне нажать на кнопку добавления нового набора ресурсов и выбрать Add Image Sets.Далее откроется окно с прямоугольными областями для эскизов изображений. Нажать в одной из прямоугольных областей на кнопку в правом верхнем углу и открывшемся после этого диалоговом окне выбрать нужный файл изображения.

Использование изображений:

xaml:
```xml
<Image x:Name="MyImage" Source="Eat.png"/>
```
Код:
```csharp
var image = ImageSource.FromResource("HellpApp.Images.MetalCube.png");
MyImage.Source = image;
```
Загрузка из сети:
```csharp
image.Source = new UriImageSource 
{ 
    CachingEnabled = true,
    CacheValidity = new System.TimeSpan(2,0,0,0),
    Uri = new System.Uri("http://www.someserver/someimage.png") 
};
```

Значение Aspect:

- AspectFit: значение по умолчанию. Если стандартное изображение не вписывается в экран (например, его ширина больше ширины экрана), то оно масштабируется с сохранением аспектного отношения (отношение ширины к длине)

- Fill: растягивает изображение по ширине или длине без сохрнения аспектного отношения

- AspectFill: сохраняет аспектное отношение, но вырезает из него ту часть, которая вписывается в экран

Xaml:
```xml
<Image Source="forest.jpg" Aspect="AspectFill" />
```

## Дата

Xaml:
```xml
<Label x:Name="LabelDate" Text="Выберите дату:" FontSize="Medium" />
<DatePicker Format="D" DateSelected="DatePicker_DateSelected">
    <DatePicker.MinimumDate>6/6/2021</DatePicker.MinimumDate>
    <DatePicker.MaximumDate>1/1/2022</DatePicker.MaximumDate>
</DatePicker>
```
Код:
```csharp
private void DatePicker_DateSelected(object sender, DateChangedEventArgs e)
{
    LabelDate.Text = $"Вы выбрали: {e.NewDate.ToString("D")}";
}
```

## Время

Xaml:
```xml
<Label x:Name="LabelTime" Text="Выберите время:" FontSize="Medium" />
<TimePicker x:Name="TimePicker" Time="12:00:00" PropertyChanged="TimePicker_PropertyChanged"/>
```
Код:
```csharp
private void TimePicker_PropertyChanged(object sender, System.ComponentModel.PropertyChangedEventArgs e)
{
    LabelTime.Text = $"Вы выбрали: {TimePicker.Time}";
}
```

## Выпадающий список

Xaml:
```xml
<Label x:Name="LabelHeader" Text="Языки программирования" FontSize="Large" />
<Picker x:Name="PickerLanguages" SelectedIndexChanged="Picker_SelectedIndexChanged" >
    <Picker.Items>
        <x:String>C#</x:String>
        <x:String>C++</x:String>
        <x:String>Java</x:String>
        <x:String>PHP</x:String>
    </Picker.Items>
</Picker>
```
Код:
```csharp
private void Picker_SelectedIndexChanged(object sender, EventArgs e)
{
    LabelHeader.Text = $"Выбран язык: {PickerLanguages.Items[PickerLanguages.SelectedIndex]}";
}
```

## Слайдеры числовых значений

Кнопки:

Xaml:
```xml
<Label x:Name="LabelStepper" Text="Число: 0" FontSize="Large"/>
<Stepper Minimum="0" Maximum="10" Increment="0.5" ValueChanged="Stepper_ValueChanged" />
```
Код:
```csharp
private void Stepper_ValueChanged(object sender, ValueChangedEventArgs e)
{
    var value = e.NewValue;
    LabelStepper.Text = $"Выбрано значение: {value:F1}";
}
```

Слайдер:

Xaml:
```xml
<Label x:Name="LabelSlider" Text="Слайдер: 0" FontSize="Large" />
<Slider Minimum="0" Maximum="100" Value="50" MinimumTrackColor="Green"
        MaximumTrackColor="Gray" ThumbColor="Green" ValueChanged="Slider_ValueChanged"/>
```
Код:
```csharp
private void Slider_ValueChanged(object sender, ValueChangedEventArgs e)
{
    LabelSlider.Text = $"Значение: {e.NewValue:F1}";
}
```

Выключатели:

Xaml:
```xml
<Label Text="Переключатель" FontSize="Large" HorizontalOptions="Center" />
<Switch HorizontalOptions="Center" Toggled="Switch_Toggled" />
<Label x:Name="LabelSwitch" FontSize="Large" HorizontalOptions="Center" />
```
Код:
```csharp
private void Switch_Toggled(object sender, ToggledEventArgs e)
{
    LabelSwitch.Text = $"Значение: {e.Value}";
}
```

## Таблица

Типы ячеек. При создании таблицы мы можем использовать разные виды ячеек:

- EntryCell: представляет метку с текстовым полем для ввода данных

- SwitchCell: представляет метку с переключателем

- TextCell: две метки для вывода текста

- ImageCell: аналогична TextCell со включением изображения

- ViewCell: содержимое и формат отображения данных ячейки определяется разработчиком

Наиболее часто используемыми из них являются SwitchCell и EntryCell.

Виды таблиц. Свойство Intent определяет виды таблицчных представлений и может принимать следующие значения:

- Data: предназначен для простого отображения данных

- Form: представляет форму ввода данных, как в примере выше

- Menu: используется для вывода меню

- Settings: используется для отображения набора настроек

Xaml:
```xml
<TableView Intent="Form">
    <TableView.Root>
        <TableRoot>
            <TableSection Title="Персональные данные">
                <EntryCell x:Name="EntryCellLogin" Label="Логин:" Keyboard="Default" Placeholder="Введите логин" Completed="EntryCell_Completed" />
                <SwitchCell x:Name="SwitchCellSave" Text="Сохранить" OnChanged="SwitchCell_OnChanged" />
            </TableSection>
            <TableSection Title="Контакты">
                <EntryCell Label="Телефон:" Keyboard="Telephone" Placeholder="Введите телефон" />
                <EntryCell Label="Почта:" Keyboard="Email" Placeholder="Введите Email" />
            </TableSection>
        </TableRoot>
    </TableView.Root>
</TableView>
<Label x:Name="LabelResult" Text="Результат" />
```
Код:
```csharp
private void EntryCell_Completed(object sender, EventArgs e)
{
    LabelResult.Text = $"Введен логин: {EntryCellLogin.Text}";
}
private void SwitchCell_OnChanged(object sender, ToggledEventArgs e)
{
    LabelResult.Text = $"Выбор: {((SwitchCellSave.On) ? "включен" : "отключен")}";
}
```

## Веб

Простой веб браузер в приложении:

Xaml:
```xml
<StackLayout Orientation="Horizontal" >
    <Button Text="GO" Clicked="Button_Clicked" />
    <Entry x:Name="UrlEntry" HorizontalOptions="FillAndExpand" />
</StackLayout>
<WebView x:Name="WebView" Source="https://airwarehouse.azurewebsites.net/" VerticalOptions="FillAndExpand"/>
```
Код:
```csharp
private void Button_Clicked(object sender, EventArgs e)
{
    WebView.Source = new UrlWebViewSource { Url = UrlEntry.Text };
}
```

## Сообщения

Сообщения - DisplayAlert & DisplayActionSheet

Xaml:
```xml
<Button Text="Показ сообщения" Clicked="Button_Clicked" />
```
Код:
```csharp
private void Button_Clicked(object sender, EventArgs e)
{
    DisplayAlert("Уведомление", "Пришло новое сообщение", "ОК");
}
private async void Button_Clicked(object sender, EventArgs e)
{
    bool result = await DisplayAlert("Подтверждение", "Действительно сделать это?", "Да", "Нет");
    await DisplayAlert("Уведомление", $"Вабранный вариант: {(result ? "Да" : "Нет")}", "ОК");
}
private async void Button_Clicked_1(object sender, EventArgs e)
{
    var action = await DisplayActionSheet("Выбор", "Отмена", "Удалить", "Вконтакте", "Твиттер", "Фейсбук");
    await DisplayAlert("Уведомление", $"Вабранный вариант: {action}", "ОК");
}
```

Запросы пользователю:

Код:
```csharp
private async void Button_Clicked_2(object sender, EventArgs e)
{
    var result = await DisplayPromptAsync("Запрос", "Как ваше имя?");
    await DisplayAlert("Результат", $"Результат: {((result != null) ? result : "<null>")}", "OK");
}
```

## Таймер

Xaml:
```xml
<Button x:Name="ButtonTimer" Text="Нажми" VerticalOptions="Center" HorizontalOptions="Center" FontSize="Large" Clicked="ButtonTimer_Clicked"/>
```
Код:
```csharp
bool alive = true;
...
    Device.StartTimer(TimeSpan.FromSeconds(1), OnTimerTick);
...
private bool OnTimerTick()
{
    ButtonTimer.Text = DateTime.Now.ToString("T");
    return alive;
}
private void ButtonTimer_Clicked(object sender, EventArgs e)
{
    if (alive)
        alive = false;
    else
    {
        alive = true;
        Device.StartTimer(TimeSpan.FromSeconds(1), OnTimerTick);
    }
}
```