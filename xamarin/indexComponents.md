# Компоненты Xamarin.Forms

## Отступы

Margin, Padding

```csharp
<StackLayout Padding="0,20,0,0">
    <BoxView Color="Green" Margin="20" />
    <BoxView Color="Blue" Margin="10, 15" />
    <BoxView Color="Red" Margin="0, 20, 15, 5" />
</StackLayout>
```

## Выравнивание горизонтальное и вертикальное

Start, Center, End, Fill, StartAndExpand, CenterAndExpand, EndAndExpand, FillAndExpand

```csharp
<StackLayout Padding="0,20,0,0">
    <Label Text="Привет из Xamarin Forms" VerticalOptions="Center" HorizontalOptions="Center" />
</StackLayout>
```

## Выравнивание текста

Start, Center, End

```csharp
<StackLayout Padding="0,20,0,0">
    <Label Text="Привет из Xamarin Forms" VerticalTextAlignment="Center" HorizontalTextAlignment="Center" />
</StackLayout>
```

## Кнопки

xaml
```csharp
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
```csharp
<StackLayout>
    <Label Font="26" Text="Многострочный ввод" />
    <Editor BackgroundColor="#a5d6a7" HeightRequest="200" />
</StackLayout>
```

## Контейнер выделения

xaml
```csharp
<Frame BorderColor="Gray" BackgroundColor="#e1e1e1" CornerRadius="8">
    <Label Text="Xamarin Forms" FontSize="Large" HorizontalOptions="Center"  />
</Frame>
```

## Изображения

Добавление для Android. В проекте для Android добавить в папку Resources/Drawable. А в окне Properties установим у этого изображения Build Action: AndroidResource.

Добавление для iOS. В проекте для iOS перейдем к папке Assets Catalogs. По умолчанию в ней должен быть каталог Assets. В открывшмся окне нажать на кнопку добавления нового набора ресурсов и выбрать Add Image Sets.Далее откроется окно с прямоугольными областями для эскизов изображений. Нажать в одной из прямоугольных областей на кнопку в правом верхнем углу и открывшемся после этого диалоговом окне выбрать нужный файл изображения.

Использование изображений:

xaml:
```csharp
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
```csharp
<Image Source="forest.jpg" Aspect="AspectFill" />
```

## Дата

Xaml:
```csharp
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
```csharp
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



