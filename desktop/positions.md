# Компоновка

87

WPF Окно может содержать только один элемент. Чтоб поместить более одного элемента нужно применять контейнер и добавлять элементы уже в этот контейнер.

Идеальное окно WPF следует принципам:

- Элементы не должны иметь явно установленных размеров. Можно только устанавливать ограничения на максимальный и минимальный размер.

- Элементы не указывают свою позицию в экранных координатах. Используются размеры, пробелы, отступы, интервалы.

- Элементы компоновки разделяют доступное пространство между своими дочерними элементами.

- Контейнеры могут быть вложенными.

Если придерживаться этих принципов, то в итоге можно получить идеальный WPF-подобный клиентский интерфейс.

Все контейнеры компоновки WPF являются панелями, унаследованными от абстрактного класса System.Windows.Controls.Panel. В этом классе дополнительно реализованы три дополнительные свойства: Background - кисть для рисования фона панели; Children - коллекция находящихся в панели элементов; IsItemsHost - булевское true, если панель используется для отображения множества элементов, ассоциированных с ItemsControl.

## Свойства компоновки

Панели компоновки взаимодействуют со своими дочерними элементами через небольшой набор свойств компоновки. Все такие свойства поддерживаются всеми графическими элементами WPF.

### Выравнивание

Свойства HorizontalAlignment и VerticalAlignment определяют позиционирование дочернего элемента внутри контейнера компоновки, когда имеется дополнительное пространство. 

По умолчанию HorizontalAlignment равно Left для меток и Stretch - для кнопок и самих панелей.

Пример:

```csharp
<StackPanel>
    <Button HorizontalAlignment="Center">Center</Button>
    <Button HorizontalAlignment="Left">Left</Button>
    <Button HorizontalAlignment="Right">Right</Button>
    <Button>Full</Button>
</StackPanel>
```

### Отступы

Свойство Margin является экземпляром структуры Tickeness, с отдльными компонентами для верхней, нижней, левой и правой граней.

При установке отступов для элементов допускается устанавливать одинаковое значение для всех сторон.

Пример:

```csharp
<Button Margin="5">Center</Button>
```

Допускается устанавливать значение отступа для каждой стороны элемента управления в порядке левое, верхнее, правое и нижнее.

Пример:

```csharp
<Button Margin="5,10,5,15">Center</Button>
```

### Минимальные, максимальные и явные размеры

Вместо установки свойствами Htight и Width явных размеров следует использовать свойства минимальных и максимальных размеров, для фиксирования элемента управления в нужных пределах размеров. При явных заданных размерах возникает риск хрупкой компоновки, не адаптирующейся к своему содержимому и усекающей его.

Пример:

```csharp
<StackPanel Margin="3">
    <Button Margin="3" HorizontalAlignment="Center">Center</Button>
    <Button Margin="3" MaxWidth="200" MinWidth="100">One</Button>
    <Button Margin="3" MaxWidth="200" MinWidth="100">Two</Button>
    <Button Margin="3" MaxWidth="200" MinWidth="100">Three</Button>
</StackPanel>
```

Для окна можно установить свойство SizeToContent в значение "WidthAndHeight", а свойства Width и Height не устанавливать - тогда окно будет автоматически устанавливать свой размер, чтобы уместить все содержимое. Если только в одно измерении, то нужно установить значение "Width" или "Height".

## Элемент Border

Удобный элемент, который принимает одну порцию вложенного содержимого и добавляет фон или рамку вокруг него. Фон, цвет рамки, толщину рамки, скругление углов рамки, пространство между рамкой и содержимым можно установить в свойствах этого элемента.

Пример:

```csharp
<Border Margin="13" Padding="16" Background="Yellow" BorderBrush="SteelBlue" BorderThickness="3,4,5,6" CornerRadius="3" VerticalAlignment="Top">
    <StackPanel Margin="3">
        <Button Margin="3" MaxWidth="200" MinWidth="100">One</Button>
        <Button Margin="3" MaxWidth="200" MinWidth="100">Two</Button>
        <Button Margin="3" MaxWidth="200" MinWidth="100">Three</Button>
    </StackPanel>
</Border>
```

## StackPanel

Элемент управления StackPanel укладывает свои дочерние элементы вдоль одиночной строки, которая может быть ориентирована горизонтально или вертикально (по умолчанию) в зависимости от значения, присвоенного свойству Orientation. 

StackPanel не пытается переносить содержимое при изменении размера окна пользователем. Взамен элементы в StackPanel просто растягиваются (согласно выбранной ориентации), приспосабливаясь к размеру самой панели StackPanel.

Панель StackPanel как правило используется как вложенная панель в какой-нибудь главной панели.

```csharp
<StackPanel Background="LightSteelBlue" Orientation ="Vertical">
    <Label Name="LabelInstruction1" FontSize="15" Content="Instruction1"/>
    <Label Name="LabelInstruction2" FontSize="15" Content="Instruction2"/>
</StackPanel>
```

## WrapPanel

Размещает элементы в доступном пространстве по одной строке или колонке за раз.

Панель WrapPanel позволяет определять содержимое, которое будет протекать сквозь панель, когда размер окна изменяется. При позиционировании элементов внутри WrapPanel их координаты верхнего левого и правого нижнего углов не указываются, как обычно делается в Canvas. Однако для каждого подэлемента допускается определение значений свойств Height и Width.

Панель WrapPanel (как и ряд других типов панелей) может быть объявлена с указанием значений ItemWidth и ItemHeight, которые управляют стандартным размером каждого элемента.

WrapPanel будет подэлементом панели другого типа, позволяя небольшой области окна переносить свое содержимое при изменении размера.

```csharp
<WrapPanel Background="LightSteelBlue">
    <Label x:Name="LabelInstruction" Width="328" Height="25" FontSize="15" Content="Instruction"/>
</WrapPanel>
```

## Позиционирование внутри панелей DockPanel

Выравнивает элементы вдоль одной из внешних границ.

Панель DockPanel обычно применяется в качестве контейнера, который содержит любое количество дополнительных панелей для группирования связанного содержимого. Панели DockPanel используют синтаксис присоединяемых свойств для управления местом, куда будет пристыковываться каждый элемент внутри DockPanel.

```csharp
<DockPanel LastChildFill="True" Background="AliceBlue">
    <Label DockPanel.Dock="Left" Name="LabelMake" Content="Make"/>
    <Button Name="ButtonOK" Content="OK"/>
</DockPanel>
```

## Позиционирование внутри панелей Grid

Выравнивает содержимое в строки или колонки невидимой таблицы. Аналогично таблице HTML панель Grid может состоять из набора ячеек, каждая из которых имеет свое содержимое.

При определении Grid нужно определить и сконфигурировать каждую колонку, строку и назначить содержимое каждой ячейки сетки с применением синтаксиса присоединяемых свойств. Определение колонок и строк выполняются с использованием элементов Grid.ColumnDefinitions и Grid.RowDefinitions. Они содержат коллекции элементов ColumnDefinition и RowDefinition соответственно.

Для помещения индивидуальных элементов в таблицу используются присоединяемые свойства Row и Column. Панели Grid также способны поддерживать разделители типа GridSplitter.

```csharp
<Grid ShowGridLines="True" Background ="LightSteelBlue">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="l*" />
        <ColumnDefinition Width="3*" />
    </Grid.ColumnDefinitions>
    <Label х:Name="LabelInstruction1" Grid.Column ="0" Grid.Row ="0" FontSize="T5" Content="Left"/>
    <GridSplitter Grid.Column ="0" Width ="5"/>
    <Label х:Name="LabelInstruction2" Grid.Column ="0" Grid.Row ="0" FontSize="T5" Content="Right"/>
</Grid>
```

Если установить свойство ShowGridLines в true, то можно будет увидеть границы между колонками и строками.

### Тонкости настройки Grid

Задавать размеры столбцов и строк в панели Grid можно одним из трех способов:

- установка абсолютных размеров (например, 100);

- установка автоматических размеров (например, Auto);

- установка пропорциональных размеров (например, * или 3*).

Пример:

```csharp
<Grid Background="LightSteelBlue">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"></RowDefinition>
        <RowDefinition Height="*"></RowDefinition>
        <RowDefinition Height="Auto"></RowDefinition>
    </Grid.RowDefinitions>
    <TextBox Margin="10" Grid.Row="0">Текстовый блок 1</TextBox>
    <TextBox Margin="10" Grid.Row="1">Текстовый блок 2</TextBox>
    <StackPanel Grid.Row="2" HorizontalAlignment="Right" Orientation="Horizontal">
        <Button Margin="10,10,5,15" Padding="5">OK</Button>
        <Button Margin="5,10,10,15" Padding="5">Отмена</Button>
    </StackPanel>
</Grid>
```

У элементов при их компоновке значения позиционирования и размерность может иметь вещественное значение, тогда происходит округление чисел. При этом может происходить размытие границ элементов. Для решени этой проблемы следует установить свойство UseLayoutRendering в значение true. Тогда WPF будет размещать границы этого элемента четко по ближайшим границам пикселей.

Пример:

```csharp
<Grid UseLayoutRendering="true">
</Grid>
```

Для объединения соседник строк и колонок следует использовать присоединенные свойства RowSpan и ColumnSpan. Эти свойства устанавливают количество строк или колонок, которые должна занять ячейка.

Пример:

```csharp
<Grid Background ="LightSteelBlue">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"></RowDefinition>
        <RowDefinition Height="Auto"></RowDefinition>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*"></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <TextBox Margin="10" Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="3">Текстовый блок 1</TextBox>
    <Button Margin="10,10,5,15" Padding="5" Grid.Row="1" Grid.Column="1">OK</Button>
    <Button Margin="5,10,10,15" Padding="5" Grid.Row="1" Grid.Column="2">Отмена</Button>
</Grid>
```

### Разделительные полосы Grid

Разделительные полосы в WPF представлены классом GridSplitter и являются средствами Grid. GridSplitter должен быть помщен в ячейку Grid. GridSplitter всегда изменяет размер всей строки или всей колонки, нужно растягивать GridSplitter по всей строке или колонке, а не ограничиваться единственной ячейкой. Нужно обязательно указать размер для GridSplitter, иначе его не будет видно. 

В случае вертикальной разделительной полосы нужно установить VerticalAlignment в Stretch, Width - фиксированный размер, а в случае горизонтальной разделительной полосы - HorizontalAlignment в Stretch, Heidht - фиксированный размер. В случае горизонтальной разделительной полосы следует установить VerticalAlignment в Center, а в случае вертикальной полосы - HorizontalAlignment в Center.

Если установить значение свойства ShowPreview в true, то при перетаскивании отображается лишь серая тень, показывающая куда попадет разделитель после отпускания курсора мышки.

Если установить значение свойства DragIncrement в какое-либо значение, то изменение позиции разделителя будет производится шагами.

Пример с одним разделителем:

```csharp
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition></RowDefinition>
        <RowDefinition></RowDefinition>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition MinWidth="50"></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
        <ColumnDefinition MinWidth="30"></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Button Grid.Row="0" Grid.Column="0">Первая</Button>
    <Button Grid.Row="0" Grid.Column="2">Вторая</Button>
    <Button Grid.Row="1" Grid.Column="0">Третья</Button>
    <Button Grid.Row="1" Grid.Column="2">Четвертая</Button>
    <GridSplitter Grid.Row="0" Grid.Column="1" Grid.RowSpan="2" Width="3" VerticalAlignment="Stretch" HorizontalAlignment="Center" ShowsPreview="true" DragIncrement="10"/>
</Grid>
```

Можно использовать несколько вертиальных/горизонтальных разделителей в одном компоненте Grid.

Пример с двумя разделителями:

```csharp
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
        <ColumnDefinition></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Grid Grid.Column="0" VerticalAlignment="Stretch">
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <Button Grid.Row="0">Первая</Button>
        <Button Grid.Row="1">Третья</Button>
    </Grid>
    <GridSplitter Grid.Column="1" Width="5" HorizontalAlignment="Center" VerticalAlignment="Stretch"/>
    <Grid Grid.Column="2" VerticalAlignment="Stretch">
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
            <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <Button Grid.Row="0">Вторая</Button>
        <Button Grid.Row="2">Четвертая</Button>
        <GridSplitter Grid.Row="1" Height="3" VerticalAlignment="Center" HorizontalAlignment="Stretch"/>
    </Grid>
</Grid>
```

Можно элементы группировать в группы с общими размерами.

Пример:

```csharp
<Grid Grid.Row="0">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" SharedSizeGroup="Test"></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
        <ColumnDefinition></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Button Grid.Column="0">Первая</Button>
    <Button Grid.Column="1">Вторая</Button>
    <Button Grid.Column="2">Третья</Button>
</Grid>
<Grid Grid.Row="1">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto" SharedSizeGroup="Test"></ColumnDefinition>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Button Grid.Column="0">Первая</Button>
    <Button Grid.Column="1">Вторая</Button>
</Grid>
```

## Позиционирование внутри UniformGrid

Этот элемент не требует предопределенных колонок и строк. Устанавливается количество колонок и строк и все пространство делится между ними поровну. Помещает элементы в невидимуют таблицу, устанавливая одинаковый размер для всех ячеек.

Пример:

```csharp
<UniformGrid Rows="2" Columns="3">
    <Button>Первая</Button>
    <Button>Вторая</Button>
    <Button>Третья</Button>
    <Button>Четвертая</Button>
</UniformGrid>
```

## Позиционирование внутри панелей Canvas

Панель Canvas делает возможным абсолютное позиционирование содержимого пользовательского интерфейса.

Для каждого элемента управления необходимо указать левый верхний угол с использованием свойств Canvas.Тор и Canvas.Left. Правый нижний угол каждого элемента управления можно задать неявно, устанавливая свойства Height и Width. Значения задаются в независимых от устройства еденицах. Альтернативно вместо Canvas.Top или Canvas.Right можно использовать Canvas.Right и Canvas.Bottom.

Наилучшим применением типа Canvas является позиционирование графического содержимого.

Пример:

```csharp
<Canvas Background="LightsteelBlue">
    <Button x:Name="ButtonOne" Canvas.Left="17" Canvas.Top="155" Width="200" Height="100" Content="Button1"/>
    <Button x:Name="ButtonTwo" Canvas.Left="120" Canvas.Top="200" Width="150" Height="50" Content="Button2"/>
</Canvas>
```

### Z-порядок

С помощью присоединяемого свойства Canvas.ZIndex можно управлять расположением элементов внутри Canvas, когда они перекрываются. По умолчанию у каждого элемента XIndex неявно равен 0.

Пример:

```csharp
<Canvas Background="LightsteelBlue">
    <Button x:Name="ButtonOne" Canvas.ZIndex="1" Canvas.Left="17" Canvas.Top="155" Width="200" Height="100" Content="Button1"/>
    <Button x:Name="ButtonTwo" Canvas.Left="120" Canvas.Top="200" Width="150" Height="50" Content="Button2"/>
</Canvas>
```

## Прокрутка

В инфраструктуре WPF поставляется класс ScrollViewer, который обеспечивает автоматическое поведение прокрутки данных внутри объектов панелей.

```csharp
<ScrollViewer>
    <StackPanel>
        <Button Content="Первая" Background="Green" Height="50"/>
    </StackPanel>
</ScrollViewer>
```












