# WPF Рисунки (кисти)

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















