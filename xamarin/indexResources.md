# Ресурсы Xamarin.Forms

## Статически Ресурсы

У каждого компонета есть свойство Resources, куда и помещается словарь ресурсов ResourceDictionary.

Каждый ресурс должен иметь ключ, задаваемый с помощью атрибута x:Key. Свойство Key через ключ ресурса будет ссылаться на данный ресурс.

Ресурсы могут определяться на трех уровнях:

На уровне отдельного элемента управления. Такие ресурсы могут применяться ко всем вложенным элементам, которые определены внутри этого элемента

На уровне всей страницы. Такие ресурсы могут применяться ко всем элементам на странице

На уровне всего приложения. Эти ресурсы доступы из любого места и из любой страницы приложения.

```xml
<ContentPage ...>
  <ContentPage.Resources>
    <ResourceDictionary>
      <Color x:Key="textColor">#004D40</Color>
      <Color x:Key="backColor">#80CBC4</Color>
      <x:Double x:Key="fontSize">24</x:Double>
    </ResourceDictionary>
  </ContentPage.Resources>
  <StackLayout>
    <Button Text="iOS" TextColor="{StaticResource Key=textColor}"
      BackgroundColor="{StaticResource Key=backColor}" FontSize="{StaticResource Key=fontSize}" />
    <Button Text="Android" TextColor="{StaticResource Key=textColor}"
      BackgroundColor="{StaticResource Key=backColor}" FontSize="{StaticResource Key=fontSize}" />
  </StackLayout>
</ContentPage>
```

С помощью ресурсов можно задать легко управлять визуальными параметрами (например, цветом, отступами и т.д.), которые должны быть специфичными для отдельным платформ.

```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <OnPlatform x:Key="textColor"
            x:TypeArguments="Color"
            iOS="Red"
            Android="Green"
            WinPhone="Blue" />
        <Color x:Key="backColor">#e1e1e1</Color>
        <x:Double x:Key="fontSize">22</x:Double>
    </ResourceDictionary>
</ContentPage.Resources>
```

Для определения общих для всего приложения ресурсов в проекте присутствует файл App.xaml.

App.xaml
```xml
<Application ...>
    <Application.Resources> 
        <ResourceDictionary>
            <Color x:Key="textColor">#004D40</Color>
            <Color x:Key="backColor">#80CBC4</Color>
            <x:Double x:Key="fontSize">22</x:Double>
        </ResourceDictionary> 
    </Application.Resources>
</Application>
```

## Динамические Ресурсы

Ресурсы могут быть статическими и динамическими.

```xml
<Button Text="Изменить" Clicked="ColorChange" TextColor="{DynamicResource Key=textColor}" />

private void ColorChange(object sender, EventArgs e)
{
    Color textColor = (Color)Resources["textColor"];
    Resources["textColor"] = textColor == Color.Red ? Color.Green : Color.Red;
}
```

## Стили

Применение стилей:

xaml
```xml
<ContentPage ...>
  <ContentPage.Resources>
    <ResourceDictionary>
      <Style x:Key="buttonStyle" TargetType="Button">
        <Setter Property="TextColor">
            <Setter.Value>
                <Color>
                    <x:Arguments>
                        <x:Double>0</x:Double>
                        <x:Double>0.75</x:Double>
                        <x:Double>0.25</x:Double>
                    </x:Arguments>
                </Color>
            </Setter.Value>
        </Setter>
        <Setter Property="BackgroundColor" Value="#80CBC4" />
        <Setter Property="FontSize" Value="Large" />
      </Style>
    </ResourceDictionary>
  </ContentPage.Resources>
  <StackLayout>
    <Button Text="iOS" Style="{StaticResource buttonStyle}" />
    <Button Text="Android" Style="{DynamicResource buttonStyle}" />
    <Button Text="Android" />
  </StackLayout>
</ContentPage>
```

Переопределение стилей:

xaml
```xml
<ContentPage ...>
    <ContentPage.Resources>
        <ResourceDictionary>
            <Style x:Key="buttonStyle" TargetType="Button">
                <Setter Property="TextColor" Value="#004D40" />
                <Setter Property="BackgroundColor" Value="#80CBC4" />
                <Setter Property="FontSize" Value="Large" />
            </Style>
        </ResourceDictionary>
    </ContentPage.Resources>
    <StackLayout>
        <Button Text="Android" Style="{StaticResource buttonStyle}" TextColor="Red" />
    </StackLayout>
</ContentPage>
```

Наследование стилей:

Свойство BasedOn позволяет наследоватся от базового другого стиля.

xaml
```xml
<ContentPage ...>
  <ContentPage.Resources>
    <ResourceDictionary>
      <Style x:Key="baseButtonStyle" TargetType="Button">
        <Setter Property="FontSize" Value="Large" />
        <Setter Property="FontFamily" Value="Verdana" />
        <Setter Property="TextColor" Value="Red" />
      </Style>
      <Style x:Key="greenButtonStyle" TargetType="Button" BasedOn="{StaticResource Key=baseButtonStyle}">
        <Setter Property="TextColor" Value="#004D40" />
        <Setter Property="BackgroundColor" Value="#80CBC4" />
      </Style>
    </ResourceDictionary>
  </ContentPage.Resources>
  <StackLayout>
    <Button Text="iOS" Style="{StaticResource Key=greenButtonStyle}" />
    <Button Text="Android" Style="{StaticResource Key=greenButtonStyle}" />
  </StackLayout>
</ContentPage>
```

Xamarin Forms предоставляет ряд встроенных стилей, которые мы можем использовать, только для элемента Label.

- BodyStyle: стиль для текста страницы

- CaptionStyle: стиль для подписей к изображениям и т.д.

- ListItemDetailTextStyle: стиль для детального описания в списке

- ListItemTextStyle: стиль для текста элемента списка

- SubtitleStyle: стиль для подзаголовка

- TitleStyle: стиль для заголовка страницы

xaml
```xml
<Label Text="Заголовок" Style="{DynamicResource TitleStyle}" />
```

## Триггеры

Триггеры в Xamarin Forms позволяют декларативно задать некоторые действия, которые выполняются при изменении свойств визуального объекта.

- Property: свойство, на изменение которого должен отслеживать триггер

- Value: значение свойства, при котором должен срабатывать триггер

- TargetType: тип объектов, к которым применяется триггер

xaml
```xml
<ContentPage.Resources>
  <ResourceDictionary>
    <Style x:Key="entryStyle" TargetType="Entry">
      <Style.Triggers>
        <Trigger Property="Entry.IsFocused" Value="True" TargetType="Entry">
          <Setter Property="TextColor" Value="Red" />
        </Trigger>
      </Style.Triggers>
    </Style>
  </ResourceDictionary>
</ContentPage.Resources>
<StackLayout>
  <Entry FontSize="Large" Style="{StaticResource Key=entryStyle}" />
</StackLayout>
```

Триггер события

Код вызываемого действия триггером:
```csharp
public class EntryValidation : TriggerAction<Entry>
{
    protected override void Invoke(Entry sender)
    {
        int number;
        if (!Int32.TryParse(sender.Text, out number))
            sender.BackgroundColor = Color.Red;
        else
            sender.BackgroundColor = Color.Default;
    }
}
```
xaml
```xml
<ContentPage ...
             xmlns:local="clr-namespace:HelloApp;assembly=HelloApp"
             ...>
  <ContentPage.Resources>
    <ResourceDictionary>
      <Style x:Key="entryStyle" TargetType="Entry">
        <Style.Triggers>
          <EventTrigger Event="TextChanged">
            <local:EntryValidation />
          </EventTrigger>
        </Style.Triggers>
      </Style>
    </ResourceDictionary>
  </ContentPage.Resources>
  <StackLayout>
    <Entry FontSize="Large" Style="{StaticResource Key=entryStyle}" />
  </StackLayout>
</ContentPage>
```

## Стили

В Xamarin Forms для стилизации элементов управления можно использовать стили CSS. 

При определении стилей в Xamarin Forms в целом мы можем использовать большинство селекторов, которые принимаются в стандартном CSS. В тоже время есть некоторые отличия.

- * Устанавливает стили для всех элементов

- element Используя название типа элемента, мы можем применить к нему стили.

- ^корневой_элемент С помощью селектора ^корневой_элемент можно определить стили для всех элементов, который расположены в данном элементе.

- #id Идентификатор элемента задается с помощью свойства StyleId.

- .class Класс элемента задается с помощью свойства StyleClass.

- element element С помощью селектора типа родительский_элемент дочерний_элемент можно задать стили для дочернего элемента, который располагается в определенном родительском элементе.

- element>element Если необходимо определим стиль только для прямых дочерних элементов, корый располагаются непосредственно в родительском элементе.

- element, element Устанавливает стили для ряда элементов.

- element+element Устанавливает стили для элемента, который идет сразу за определенным элементом.

- element~element Устанавливает стили для элемента, который идет непосредственно перед определенным элементом.

Стилевые свойства:

- background-color Представляет свойство BackgroundColor. Применяется к элементу типа VisualElement.

- background-image Представляет свойство BackgroundImage. Применяется к элементу типа Page.

- border-color Представляет свойство BorderColor. Применяется к элементам типа Button и Frame.

- border-width Представляет свойство BorderWidth. Применяется к элементу Button. Принимает значение типа double.

- color Представляет свойство TextColor (цвет текста). Применяется к элементам Button, DatePicker, Editor, Entry, Label, Picker, SearchBar, TimePicker.

- font-family

Соответствует свойству FontFamily. Применяется к элементам Button, DatePicker, Editor, Entry, Label, Picker, SearchBar, TimePicker, Span.

- font-size

Соответствует свойству FontSize. Применяется к элементам Button, DatePicker, Editor, Entry, Label, Picker, SearchBar, TimePicker, Span.

В качестве значения могут использоваться значения типа double или именованные размеры (small, large, medium).

- font-style

Соответствует свойству FontStyle. Применяется к элементам Button, DatePicker, Editor, Entry, Label, Picker, SearchBar, TimePicker, Span.

Допустимые значения: bold (выделение жирным) и italic (выделение курсивом).

- height

Соответствует свойству HeightRequest. Применяется к элементу VisualElement. Принимает значение типа double.

- width

Соответствует свойству WidthRequest. Применяется к элементу VisualElement. Принимает значение типа double.

- margin

Соответствует свойству Margin. Применяется к элементу View. Принимает значение типа Thickness.

- padding Соответствует свойству Padding. Применяется к элементу Layout, Page. Принимает значение типа Thickness.

- text-align Соответствует свойству HorizontalTextAlignment (выравнивание текста). Применяется к элементам Entry, EntryCell, Label, SearchBar. Допустимые значения: left, right, center, start, end.

- visibility Соответствует свойству IsVisible. Применяется к элементу VisualElement. Допустимые значения: true, visible, false, hidden, collapse.

- opacity Соответствует свойству Opacity. Применяется к элементу VisualElement. Допустимые значения: true, visible, false, hidden, collapse.

thickness:

- Одно значение задает отступ для всех сторон

- Два значения задают отступы соответственно по вертикали и по горизонтали

- Три значения задают соответственно верхний отступ, отступ по горизонтали (как справа, так и слева) и отступ снизу

- Четыре значения задают соответственно отступ сверху, справа, снизу и слева

Файл стилей styles.css в папке styles:
```css
^contentpage {
    background-color: darkseagreen;
}
stacklayout {
    margin: 15;
    padding: 10;
}
label {
    color: black;
}
#header {
    font-size: 32;
    font-weight: bold;
}
.english {
    font-weight: bold;
    font-size: large;
    color: darkblue;
}
.russian {
    font-size: medium;
}
stacklayout label {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
}
```

Использование стилей, xaml:
```xml
<ContentPage.Resources>
    <StyleSheet Source="/styles/styles.css" />
</ContentPage.Resources>

<StackLayout Padding="5">
    <Label x:Name="Header" StyleId="header" Text="Заголовок" />
    <Label Text="apple" StyleClass="english" />
    <Label Text="яблоко" StyleClass="russian" />
    <Label Text="house" StyleClass="english" />
    <Label Text="дом" StyleClass="russian" />

</StackLayout>
```

