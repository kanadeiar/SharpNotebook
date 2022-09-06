# Фигуры

В инфраструктуре WPF используется разновидность графической визуализации под названием графика режима сохранения и программистам не приходится самим писать и сопровождать гораздо меньший объем рутиного кода для поддержки графики.

Существует аспект выбора при графике WPF как делать графику - в XAML или в C#. Три подхода:

- Фигуры. Пространство имен System.Windows.Shapes определяет небольшое количество классов для визуализации двумерных объектов. Мощные, но в случае множественного применения могут привести к значительным накладным расходам памяти.

- Рисунки и геометрические объекты. Предполагает работу с классами, производными от абстрактного класса System.Windows.Media.Drawing.

- Визуальные объекты. Самый быстрый и легковесный способ визуализауии графики в WPF дает работу с визуальным уровнем только через код C#. Через System.Windows.Media.Visual можно взаимодействовать с WPF.

Если нужно умеренный объем интерактивных графических данных, то нужно применять System.Windows.Shapes.

Для сложных и по большей части не интерактивных векторных графических данных лучше всего подходят рисунки и геометрические объекты - через XAML и C#.

Для реализации быстрого способа визуализации значительных объемов графических данных, то нужно выбрать визуальный уровень. Но это только из кода C#.

## Фигуры

Оно дает наиболее прямолинейный, интерактивный и самый затратный в плане расхода памяти способ визуализации. Сборка - PresentationFramework.dll содержит шесть запечатанных классов, расширяющих Shape: Ellipse, Rectangle, Line, Polygon, Polyline, Path.

Пример интерактивного визуального элемента:

```csharp
<Rectangle x:Name="myRect" Height="30" Width="30" Fill="Green" MouseDown="MyRect_OnMouseDown" MouseUp="MyRect_OnMouseUp"/>
private void MyRect_OnMouseDown(object sender, MouseButtonEventArgs e)
{
    myRect.Fill = Brushes.Beige;
}
private void MyRect_OnMouseUp(object sender, MouseButtonEventArgs e)
{
    myRect.Fill = Brushes.Aquamarine;
}
```

Пример рисования прямоугольников, эллипсов и линий:

```csharp
<Window x:Class="WpfApp1.MainWindow"
...
        Title="MainWindow" Height="400" Width="600">
    <DockPanel>
        <ToolBar DockPanel.Dock="Top" Name="mainToolBar" Height="60">
            <RadioButton Name="circleOption" GroupName="shapeSelection" Click="CircleOption_OnClick">
                <Ellipse Fill="Green" Height="40" Width="40"/>
            </RadioButton>
            <RadioButton Name="rectOption" GroupName="shapeSelection" Click="RectOption_OnClick">
                <Rectangle Fill="Red" Height="40" Width="40" RadiusX="10" RadiusY="10"/>
            </RadioButton>
            <RadioButton Name="lineOption" GroupName="shapeSelection" Click="LineOption_OnClick">
                <Line Height="40" Width="40" Stroke="Blue" StrokeThickness="10" X1="5" Y1="5" X2="35" Y2="35" StrokeStartLineCap="Round" StrokeEndLineCap="Round"/>
            </RadioButton>
        </ToolBar>
        <Canvas Background="LightYellow" Name="canvasDrawingArea" MouseDown="CanvasDrawingArea_OnMouseDown" MouseRightButtonDown="CanvasDrawingArea_OnMouseRightButtonDown"/>
    </DockPanel>
</Window>
private enum SelectShape
{ Circle, Rectangle, Line }
private SelectShape currentShape; //выбранная фигура рисования
public MainWindow()
{
    InitializeComponent();
}
private void CircleOption_OnClick(object sender, RoutedEventArgs e)
{
    currentShape = SelectShape.Circle; //выбрать фигуру рисования
}
private void RectOption_OnClick(object sender, RoutedEventArgs e)
{
    currentShape = SelectShape.Rectangle;
}
private void LineOption_OnClick(object sender, RoutedEventArgs e)
{
    currentShape = SelectShape.Line;
}
private void CanvasDrawingArea_OnMouseDown(object sender, MouseButtonEventArgs e)
{
    if (e.ChangedButton == MouseButton.Right)
        return; //правая кнопка не рисует фигуру
    Shape shape = null;
    switch (currentShape)
    {
        case SelectShape.Circle:
            shape = new Ellipse
            {
                Fill = Brushes.Green, Height = 40, Width = 40,
            };
            break;
        case SelectShape.Rectangle:
            shape = new Rectangle
            {
                Fill = Brushes.Red, Height = 40, Width = 40, RadiusX = 10, RadiusY = 10,
            };
            break;
        case SelectShape.Line:
            shape = new Line
            {
                Stroke = Brushes.Blue, StrokeThickness = 10, X1 = 5, Y1 = 5, X2 = 35, Y2 = 35,
                StrokeStartLineCap = PenLineCap.Round, StrokeEndLineCap = PenLineCap.Round,
            };
            break;
        default:
            return;
    }
    Canvas.SetLeft(shape, e.GetPosition(canvasDrawingArea).X);
    Canvas.SetTop(shape, e.GetPosition(canvasDrawingArea).Y);
    canvasDrawingArea.Children.Add(shape);
}
private void CanvasDrawingArea_OnMouseRightButtonDown(object sender, MouseButtonEventArgs e)
{
    Point point = e.GetPosition((Canvas)sender);
    HitTestResult result = VisualTreeHelper.HitTest(canvasDrawingArea, point);
    if (result != null) //правая кнопка удаляет фигуру
    {
        canvasDrawingArea.Children.Remove(result.VisualHit as Shape);
    }
}
```

Пример ломаной линии и полигона:

```csharp
<Canvas Background="LightYellow" Name="canvasDrawingArea" MouseDown="CanvasDrawingArea_OnMouseDown" MouseRightButtonDown="CanvasDrawingArea_OnMouseRightButtonDown">
    <Polyline Stroke="BlueViolet" StrokeThickness="20" StrokeLineJoin="Round" Points="10,10 40,40 10,90 300,20"/>
    <Polygon Fill="AliceBlue" StrokeThickness="5" Stroke="Green" Points="140,110 180,180 60,150"/>
</Canvas>
```

Графический элемент Path способен определять сложный двухмерный графический объект в виде коллекции независимых геометрических объектов.

Пример фигуры:

```csharp
<Canvas Background="LightYellow" Name="canvasDrawingArea" MouseDown="CanvasDrawingArea_OnMouseDown" MouseRightButtonDown="CanvasDrawingArea_OnMouseRightButtonDown">
    <Path Fill="Orange" Stroke="BlueViolet" StrokeThickness="3">
        <Path.Data>
            <CombinedGeometry GeometryCombineMode="Union">
                <CombinedGeometry.Geometry1>
                    <EllipseGeometry Center="75,75" RadiusX="40" RadiusY="40"/>
                </CombinedGeometry.Geometry1>
                <CombinedGeometry.Geometry2>
                    <RectangleGeometry Rect="15,60 120,30"/>
                </CombinedGeometry.Geometry2>
            </CombinedGeometry>
        </Path.Data>
    </Path>
</Canvas>
```

Графические элемент PathGeometry наиболее сложен для конфигурирования в терминах XAML и кода. Каждый сегмент PathGeometry состоит из объектов, содержащих разнообразные сегменты и фигуры. 

Пример:

```csharp
<Canvas Background="LightYellow" Name="canvasDrawingArea" MouseDown="CanvasDrawingArea_OnMouseDown" MouseRightButtonDown="CanvasDrawingArea_OnMouseRightButtonDown">
    <Path Fill="Orange" Stroke="BlueViolet" StrokeThickness="3">
        <Path.Data>
            <PathGeometry>
                <PathGeometry.Figures>
                    <PathFigure StartPoint="10,10">
                        <PathFigure.Segments>
                            <BezierSegment Point1="100,0" Point2="200,200" Point3="300,100"/>
                            <LineSegment Point="400,100"/>
                            <ArcSegment Size="50,50" RotationAngle="45" IsLargeArc="True" SweepDirection="Clockwise" Point="200,100"/>
                        </PathFigure.Segments>
                    </PathFigure>
                </PathGeometry.Figures>
            </PathGeometry>
        </Path.Data>
    </Path>
</Canvas>
```

Можно вместо установки в свойство Data объекта Path коллекции объектов можно устанавливать одиночный строковый литерал, содержащий набор символов и различных значений, определяющих фигуру:

```csharp
<Canvas Background="LightYellow" Name="canvasDrawingArea" MouseDown="CanvasDrawingArea_OnMouseDown" MouseRightButtonDown="CanvasDrawingArea_OnMouseRightButtonDown">
    <Path Fill="Orange" Stroke="BlueViolet" StrokeThickness="3" Data="M 10,10 310,100 350,160">
    </Path>
</Canvas>
```

## Графические трансформации

Трансформации могут применяться к любым объектам пользовательского интерфейса. Можно ее назначить с помощью двух общих свойств: 

- LayoutTransform - трансформация происзодит перед визуализацией элементов в диспетчере компоновки и 

- RenderTransform - инициируется после того, как элементы попали в свои контейнеры, потому возможно что будут трансформированы с перекрытием друг друга.

Примеры применения трансформации:

```csharp
<Rectangle Height="100" Width="40" Fill="Red">
    <Rectangle.LayoutTransform>
        <RotateTransform Angle="45"/>
    </Rectangle.LayoutTransform>
</Rectangle>
<Button Content="Нажми меня" Width="100" Height="40">
    <Button.LayoutTransform>
        <SkewTransform AngleX="20" AngleY="20"/>
    </Button.LayoutTransform>
</Button>
<Ellipse Fill="Blue" Width="5" Height="5">
    <Ellipse.LayoutTransform>
        <ScaleTransform ScaleX="20" ScaleY="20"/>
    </Ellipse.LayoutTransform>
</Ellipse>
<TextBox Text="Раз" Width="50" Height="50">
    <TextBox.LayoutTransform>
        <TransformGroup>
            <RotateTransform Angle="45"/>
            <SkewTransform AngleX="-20" AngleY="-20"></SkewTransform>
        </TransformGroup>
    </TextBox.LayoutTransform>
</TextBox>
```

Пример трансформации применительно к диспетчеру компоновки, чтоб трансформировать все внутренние данные:

```csharp
<DockPanel LastChildFill="True">
    <DockPanel.LayoutTransform>
        <RotateTransform Angle="45"></RotateTransform>
    </DockPanel.LayoutTransform>        
```

Пример трансформации по событию - клику мыши:

```csharp
<ToggleButton Name="flipCanvas" Click="FlipCanvas_OnClick" Content="Поворот содержимого"/>
...
private void FlipCanvas_OnClick(object sender, RoutedEventArgs e)
{
    if (true == flipCanvas.IsChecked)
    {
        RotateTransform rotate = new RotateTransform(-180);
        canvasDrawingArea.LayoutTransform = rotate;
    }
    else
    {
        canvasDrawingArea.LayoutTransform = null;
    }
}
```

Свойство у визуального элемента Canvas, позволяющее предотвращать визуализауию дочерних элементов вне границ родительского элемента:

```csharp
<Canvas ClipToBounds="True" ... />
//перед установкой элементов на холст - повернуть их
if (true == flipCanvas.IsChecked) //если повернут холст, то повернуть устанавливаемую фигуру
{
    RotateTransform rotate = new RotateTransform(-180);
    shape.RenderTransform = rotate;
}
```

Примеры трансформации холста:

```csharp
...
    myCanvas.LayoutTransform = new ScaleTransform(-1, 1);
...
    myCanvas.LayoutTransform = new RotateTransform(180);
...
    myCanvas.LayoutTransform = new SkewTransform(40, -20);
...
```









