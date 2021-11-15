# Базовый Xamarin.Forms

## Базовое приложение

Проект -> Xamarin.Forms.

Текст со сложным оформлением:
```xml
<Label FontSize="16" Padding="30,24,30,0">
    <Label.FormattedText>
        <FormattedString>
            <FormattedString.Spans>
                <Span Text="Learn more at "/>
                <Span Text="https://aka.ms/xamarin-quickstart" FontAttributes="Bold"/>
            </FormattedString.Spans>
        </FormattedString>
    </Label.FormattedText>
</Label>
```

## Расширения разметки XAML

В Xamarin Forms есть ряд встроенных расширений:
```csharp
x:Static
x:Type
x:Array
x:Null
x:Reference
StaticResource
DynamicResource
Binding
ConstraintExpression
```

x:Static позволяет привязать к атрибуту значение константы, статической переменной, статического свойства или значения перечисления.

Код:
```csharp
public const string HEADER = "Xamarin";
```

Xaml:
```xml
xmlns:local="clr-namespace:HelloApp"
<Label Text="{x:Static local:MainPage.HEADER}" />
```

Расширение разметки x:Array позволяет определить массив данных.
Xaml:
```xml
<ListView Margin="20,0,20,0" SeparatorVisibility="None">
    <ListView.ItemTemplate>
        <DataTemplate>
            <ViewCell>
                <ViewCell.View>
                    <FlexLayout Direction="Row" AlignItems="Center" JustifyContent="Start" Padding="5">
                        <Label Text="{Binding}"/>
                    </FlexLayout>
                </ViewCell.View>
            </ViewCell>
        </DataTemplate>
    </ListView.ItemTemplate>
    <ListView.ItemsSource>
        <x:Array Type="{x:Type x:String}">
            <x:String>iPhone</x:String>
            <x:String>Philips</x:String>
            <x:String>Samsung</x:String>
            <x:String>Apple</x:String>
            <x:String>Mac</x:String>
            <x:String>Nokia</x:String>
            <x:String>Holo</x:String>
            <x:String>Mago</x:String>
            <x:String>Apple</x:String>
            <x:String>Mac</x:String>
            <x:String>Nokia</x:String>
            <x:String>Holo</x:String>
            <x:String>Mago</x:String>
        </x:Array>
    </ListView.ItemsSource>
</ListView>
```

## Компоновка

Для определения содержимого страницы класс страницы ContentPage имеет свойство Content. Элементы компоновки:

- StackLayout

- AbsoluteLayout

- RelativeLayout

- Grid

- FlexLayout

Xaml:
```xml
<StackLayout x:Name="stackLayout" Spacing="8">
    <Label Text="Первая метка" TextColor="Red"  />
    <Label Text="Вторая метка" TextColor="Blue" />
</StackLayout>
<StackLayout x:Name="stackLayout" Spacing="8" Orientation="Horizontal">
  <Label Text="Первая метка" TextColor="Red"  />
  <Label Text="Вторая метка" TextColor="Blue" />
</StackLayout>
```

Xaml:
```xml
<ScrollView>
    <StackLayout>
      <Label Text="Метка 1" FontSize="23" />
      <Label Text="Метка 2" FontSize="23" />
      <!--..................................................-->
      <Label Text="Метка 20" FontSize="23" />
    </StackLayout>
</ScrollView>
```

Xaml:
```xml
<AbsoluteLayout>
    <BoxView Color="LightBlue" AbsoluteLayout.LayoutBounds="70, 70, 200, 70" />
    <Label Text="Заголовок" FontSize="Large" AbsoluteLayout.LayoutBounds="110, 90, 150, 60" />
    <Label Text="Основное содержание текста" FontSize="Medium" AbsoluteLayout.LayoutBounds="70, 150, AutoSize, AutoSize" />
</AbsoluteLayout>
  <AbsoluteLayout>
    <BoxView Color="Red" AbsoluteLayout.LayoutBounds=".1, .2, 50, 80" AbsoluteLayout.LayoutFlags="PositionProportional" />
    <BoxView Color="Green" AbsoluteLayout.LayoutBounds="1, .2, 50, 80" AbsoluteLayout.LayoutFlags="PositionProportional" />
    <BoxView Color="Blue" AbsoluteLayout.LayoutBounds=".4, .8, .2, .2" AbsoluteLayout.LayoutFlags="All" />
 </AbsoluteLayout>
```

Xaml:
```xml
<RelativeLayout>
    <BoxView WidthRequest="100" HeightRequest="100" Color="Blue"
        RelativeLayout.XConstraint= "{ConstraintExpression Type=RelativeToParent, 
            Property=Width, Factor=0.5, Constant=-50}" 
        RelativeLayout.YConstraint= "{ConstraintExpression Type=RelativeToParent,
            Property=Height, Factor=0.5, Constant=-50}" />
</RelativeLayout>
<RelativeLayout>
    <Label x:Name="lbl" Text="RelativeLayout"
            RelativeLayout.XConstraint = "{ConstraintExpression Type=RelativeToParent,
            Property=Width, Factor=0.5, Constant=-50}"
            RelativeLayout.YConstraint = "{ConstraintExpression Type=RelativeToParent, 
            Property=Height, Factor=0.5, Constant=-150}" />
    <BoxView Color="Blue"
            RelativeLayout.XConstraint = "{ConstraintExpression Type=RelativeToView, ElementName=lbl,
            Property=X, Factor=1, Constant=-30}"
            RelativeLayout.YConstraint = "{ConstraintExpression Type=RelativeToView, ElementName=lbl, 
            Property=Y, Factor=1, Constant=30}"
        RelativeLayout.WidthConstraint = "150" RelativeLayout.HeightConstraint = "100"/>
</RelativeLayout>
```

Xaml:
```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Height="*" />
        <RowDefinition Height="*" />
        <RowDefinition Height="*" />
    </Grid.RowDefinitions>
    <BoxView Color="Red" Grid.Column="0" Grid.Row="0" />
    <BoxView Color="Blue" Grid.Column="0" Grid.Row="1" />
    <BoxView Color="Fuchsia" Grid.Column="0" Grid.Row="2" />

    <BoxView Color="Teal" Grid.Column="1" Grid.Row="0" />
    <BoxView Color="Green" Grid.Column="1" Grid.Row="1" />
    <BoxView Color="Maroon" Grid.Column="1" Grid.Row="2" />

    <BoxView Color="Olive" Grid.Column="2" Grid.Row="0" />
    <BoxView Color="Pink" Grid.Column="2" Grid.Row="1" />
    <BoxView Color="Yellow" Grid.Column="2" Grid.Row="2" />
</Grid>
```


