# Ресурсы Xamarin.Forms

## Статически Ресурсы

У каждого компонета есть свойство Resources, куда и помещается словарь ресурсов ResourceDictionary.

Каждый ресурс должен иметь ключ, задаваемый с помощью атрибута x:Key. Свойство Key через ключ ресурса будет ссылаться на данный ресурс.

Ресурсы могут определяться на трех уровнях:

На уровне отдельного элемента управления. Такие ресурсы могут применяться ко всем вложенным элементам, которые определены внутри этого элемента

На уровне всей страницы. Такие ресурсы могут применяться ко всем элементам на странице

На уровне всего приложения. Эти ресурсы доступы из любого места и из любой страницы приложения.

```xaml
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
```csharp
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

```csharp
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
```csharp
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
```csharp
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
```csharp
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
```csharp
<Label Text="Заголовок" Style="{DynamicResource TitleStyle}" />
```

## Триггеры

Триггеры в Xamarin Forms позволяют декларативно задать некоторые действия, которые выполняются при изменении свойств визуального объекта.

- Property: свойство, на изменение которого должен отслеживать триггер

- Value: значение свойства, при котором должен срабатывать триггер

- TargetType: тип объектов, к которым применяется триггер

xaml
```csharp
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
```xaml
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

