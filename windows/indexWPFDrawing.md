# WPF Рисунки (кисти, векторные изображения)

## Кисти

Каждый способо визуализации - фигуры, рисование и визуальные объекты интенсивоно используют кисти, управляющие заполнением внутренней области фигуры.

Класс кистей System.Windows.Media.Brush предоставляет шесть разных типов кистей:
```csharp
DrawingBrush                Заполняет область с помощью объекта (Drawing) GeometryDrawing, ImageDrawing, VideoDrawing 
OmageBrush                  Заполняет область изображением 
LinearGradientBrush         Заполняет область линейным градиентом
RadianGradientBrush         Заполняет область радиальным градиентом
SolidColorBrush             Заоплняет область сплошным цветом из Color
VisualBrush                 Заполняет область с помощью обхекта (Visual) DrawingVisual, Viewport3DVisual, ContentVisual
```
Пример пользовательской специальной кисти:
```csharp
<Ellipse Height="40" Width="40">
    <Ellipse.Fill>
        <RadialGradientBrush>
            <GradientStop Color="#FFFFF300" Offset="0"/>
            <GradientStop Color="#FF246300" Offset="0.504"/>
            <GradientStop Color="#FFB7FFC4" Offset="1"/>
        </RadialGradientBrush>
    </Ellipse.Fill>
</Ellipse>
//кисть в коде
RadialGradientBrush brush = new RadialGradientBrush();
brush.GradientStops.Add(new GradientStop((Color)ColorConverter.ConvertFromString("#FFFFF300"), 0));
brush.GradientStops.Add(new GradientStop((Color)ColorConverter.ConvertFromString("#FF246300"),0.504));
brush.GradientStops.Add(new GradientStop((Color)ColorConverter.ConvertFromString("#FFB7FFC4"),1));
shape.Fill = brush;
break;
```
Объект Color можно точно настраивать числами:
```csharp
shape = new Rectangle
{
    Height = 40, Width = 40, RadiusX = 10, RadiusY = 10,
};
Color color = new Color() {R = 20, G = 10, B = 20, A = 90};
shape.Fill = new SolidColorBrush(color);
```

## Перья

Перо:
```csharp
<Pen Thickness="10" LineJoin="Round" EndLineCap="Triangle" StartLineCap="Triangle"/>
```

## Рисование геометрических объектов

Способы рисования - классы, расширяющие абстрактный класс System.Windows.Media.Drawing:
```csharp
DrawingGroup                Комбинаци коллекции отдельных объектов, производных от Drawing, в единую визуализацию
GeometryDrawing             Применяется для визуализации фигур в легковесной манере 
GlyphRunDrawing             Визуализация текстовых данных с применением WPF
ImageDrawing                Визуализация файла изображения или набора геометрических объектов внути прямоугольника
VideoDrawing                Воспроизведение аудио и видеофайла. Тип может полноценно использоватся только в процедурном коде.
```
Производные типы от Drawing типы не умеют визуализировать себя, должны быть помещены в какой-то контейнер - DrawingImage (помещает рисунки и геометрические объекты внутрь Image), DrawingBrush(кисть на основе рисунков и геометрических объектов), DrawingVisual(визульный уроень графической визуализации, управляемым процедурным кодом). Визуализация друмерного графического изображения в итоге получается с намного меньшими накладными расходами, чем в случае визуализации посредством фигур.

Пример рисования с помощью DrawingBrush в окне:
```csharp
...
<Grid>
    <Grid.Background>
        <DrawingBrush>
            <DrawingBrush.Drawing>
                <GeometryDrawing>
                    <GeometryDrawing.Geometry>
                        <GeometryGroup>
                            <EllipseGeometry Center="70,70"/>
                            <RectangleGeometry Rect="25,50 40 10"/>
                            <LineGeometry StartPoint="0,0" EndPoint="50,40"/>
                        </GeometryGroup>
                    </GeometryDrawing.Geometry>
                    <GeometryDrawing.Pen>
                        <Pen Brush="Blue" Thickness="2"/>
                    </GeometryDrawing.Pen>
                    <GeometryDrawing.Brush>
                        <SolidColorBrush Color="Orange"/>
                    </GeometryDrawing.Brush>
                </GeometryDrawing>
            </DrawingBrush.Drawing>
        </DrawingBrush>
    </Grid.Background>
</Grid>
...
```
Пример рисования c помощью DrawingBrush на кнопке:
```csharp
<Button Height="200" Width="200">
    <Button.Background>
        <DrawingBrush>
... //тоже самое
        </DrawingBrush>
    </Button.Background>
</Button>
```
Пример рисования с помощью DrawingImage на элементе Image:
```csharp
...
<Grid>
    <Image>
        <Image.Source>
            <DrawingImage>
                <DrawingImage.Drawing>
                    <GeometryDrawing>
                        <GeometryDrawing.Geometry>
                            <GeometryGroup>
                                <EllipseGeometry Center="70,70"/>
                                <RectangleGeometry Rect="25,50 40 10"/>
                                <LineGeometry StartPoint="0,0" EndPoint="50,40"/>
                            </GeometryGroup>
                        </GeometryDrawing.Geometry>
                        <GeometryDrawing.Pen>
                            <Pen Brush="Blue" Thickness="2"/>
                        </GeometryDrawing.Pen>
                        <GeometryDrawing.Brush>
                            <SolidColorBrush Color="Orange"/>
                        </GeometryDrawing.Brush>
                    </GeometryDrawing>
                </DrawingImage.Drawing>
            </DrawingImage>
        </Image.Source>
    </Image>
</Grid>
...
```

## Векторные изображения

Прежде чем использовать сложные графические данны в приложении WPF, графику нужно преобразовать в данные путей.

Способ импорта векторного изображения из файла *.SVG в разметку XAML:

1. Программа Inkscape (inkscape.org) -> открыть файл и его распечатать на принтере -> Принтер "Microsoft XPS Document Writer".

2. Файл полученный переименовать в расширение *.zip. Открыть архив, в папке "Documents/1/Pages" будет лежать файл 1.fpage.

3. Открыть файл, скоприровать из него текст путей, вставить в область <Canvas></Canvas>.

Пример взаимодействия с векторной графикой внутри <Canvas></Canvas>:
```csharp
<Canvas MouseLeftButtonDown="UIElement_OnMouseLeftButtonDown">
    <Path Data="F1 M 4.16,9.52 L 104.24,9.52 104.24,79.52 4.16,79.52 z"  Fill="#ff00ffff" />
    <Path Data="F1 M 48.48,71.04 L 455.68,71.04 455.68,343.84 48.48,343.84 z"  Fill="#ff00ffff" />
</Canvas>
private void UIElement_OnMouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
    if (e.OriginalSource is Path p)
    {
        p.Fill = new SolidColorBrush(Colors.Red);
    }
}
```


