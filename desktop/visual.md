# Визуализация

Визуальный уровень обеспечивает самый быстрый способ визуализации крупных объемов графических данных.

Абстрактный класс Sytem.Windows.Media.Visual дает набор служб для визуализации графики, но не поддерживают службы, могущие приводить к разбуханию кода. Этот класс определяет несколько подклассов - DrawingVisual(легковесный класс рисования, используемый для рисования фигур, изображений или текста), Viewport3DVisual, ContainerVisual. 

Для визуализации каких-либо данных на поверности, минимум необходимо:

- получить объект DrawingContext из DrawingVisual.

- объектом DrawingContext визуализировать графические данные.

Но для того чтобы визуализированные графические данные реагировали на действия пользователя мышкой, нужно дополнительно:

- обновить логичесое и визуальное деревья контейнера.

- переопределить два виртуальных метода FrameworkElement для получения контейнером визуальных данных.

Пример простой визуализации данных:

```csharp
...
    Loaded="MainWindow_OnLoaded">
<StackPanel Background="AliceBlue" Name="myPanel">
    <Image x:Name="myImage">
    </Image>
</StackPanel>
...
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    //создать форматированный текст
    FormattedText text = new FormattedText("Визуализация", new CultureInfo("ru-ru"),FlowDirection.LeftToRight,new Typeface(this.FontFamily,FontStyles.Italic,FontWeights.DemiBold,FontStretches.UltraExpanded),30,Brushes.DarkGreen,null,VisualTreeHelper.GetDpi(this).PixelsPerDip);
    //создать контекст визуализации
    DrawingVisual drawingVisual = new DrawingVisual();
    using (DrawingContext drawingContext = drawingVisual.RenderOpen())
    {
        //методы DrawingContext для визуализации данных
        drawingContext.DrawRoundedRectangle(Brushes.Yellow, new Pen(Brushes.Black, 5), new Rect(5,5,400,100), 20, 20);
        drawingContext.DrawText(text, new Point(20,20));
    }
    //битовое изображение из визуального контекста
    RenderTargetBitmap bitmap = new RenderTargetBitmap(500,300,100,100,PixelFormats.Pbgra32);
    bitmap.Render(drawingVisual);
    myImage.Source = bitmap;
}
```

После создания специального диспетчера компоновки (Grid, StackPanel, Canvas ...) его можно подключить к окну и тогда часть интерфейса будет использовать высоко оптимизированный агент визуализации, а для некритичных - фигуры и рисунки.

Пример визуализации с использованием элемента StackPanel:

```csharp
<StackPanel Background="AliceBlue" Name="myPanel">
...
    <local:CustVisualFrameworkElement/>
</StackPanel>
//класс поддержки переменной - члена типа VisualCollection
public class CustVisualFrameworkElement : FrameworkElement
{
    private VisualCollection collection; //коллекция всех элементов
    public CustVisualFrameworkElement()
    {
        collection = new VisualCollection(this) //заполнить коллекцию квадартом и кругом
        {
            AddRect(),
            AddCircle(),
        };
    }
    private Visual AddCircle()
    {
        DrawingVisual drawingVisual = new DrawingVisual(); 
        using (DrawingContext drawingContext = drawingVisual.RenderOpen()) //объект для создания нового содержимого
        {
            drawingContext.DrawEllipse(Brushes.DarkBlue, null, new Point(80,80), 40,50); //создать круг и нарисовать его
        }
        return drawingVisual;
    }
    private Visual AddRect()
    {
        DrawingVisual drawingVisual = new DrawingVisual();
        using (DrawingContext drawingContext = drawingVisual.RenderOpen()) //объект для создания нового содержимого
        {
            drawingContext.DrawRectangle(Brushes.Tomato, null, new Rect(new Point(150,100), new Size(300,100))); //создать квадрат и нарисовать его
        }
        return drawingVisual;
    }
    protected override int VisualChildrenCount => collection.Count;
    protected override Visual GetVisualChild(int index)
    {
        if (index < 0 || index >= collection.Count)
        {
            throw new ArgumentOutOfRangeException();
        }
        return collection[index]; //коллекция из графических элементов
    }
}
private void MainWindow_OnLoaded(object sender, RoutedEventArgs e)
{
    //создать форматированный текст
    FormattedText text = new FormattedText("Визуализация", new CultureInfo("ru-ru"),FlowDirection.LeftToRight,new Typeface(this.FontFamily,FontStyles.Italic,FontWeights.DemiBold,FontStretches.UltraExpanded),30,Brushes.DarkGreen,null,VisualTreeHelper.GetDpi(this).PixelsPerDip);
    //создать контекст визуализации
    DrawingVisual drawingVisual = new DrawingVisual();
    using (DrawingContext drawingContext = drawingVisual.RenderOpen())
    {
        //методы DrawingContext для визуализации данных
        drawingContext.DrawRoundedRectangle(Brushes.Yellow, new Pen(Brushes.Black, 5), new Rect(5,5,400,100), 20, 20);
        drawingContext.DrawText(text, new Point(20,20));
    }
    //битовое изображение из визуального контекста
    RenderTargetBitmap bitmap = new RenderTargetBitmap(500,300,100,100,PixelFormats.Pbgra32);
    bitmap.Render(drawingVisual);
    myImage.Source = bitmap;
}
```

Проверка попадания курсором мыши. Так как класс DrawingVisual не располагает инфраструктурой UIElement или FrameworkElement, нужно программно добавить реагирование на операции проверки попадания.

Пример добавления реагирования на шелчки пользователя:

```csharp
public class CustVisualFrameworkElement : FrameworkElement
...
    public CustVisualFrameworkElement()
    {
        collection = new VisualCollection(this) //заполнить коллекцию квадартом и кругом
        {
            AddRect(),
            AddCircle(),
        };
        //дополнительно реагирование на нажатие мышки
        this.MouseDown += CustVisualFrameworkElement_MouseDown;
    }
    private void CustVisualFrameworkElement_MouseDown(object sender, System.Windows.Input.MouseButtonEventArgs e)//реагирование на нажатия мыши
    {
        Point point = e.GetPosition((UIElement)sender); //найти точку где произведен щелчок
        VisualTreeHelper.HitTest(this, null, new HitTestResultCallback(myCallback), new PointHitTestParameters(point)); //просмотреть дерево на щелчки и вызвать делегат если был щелчок
    }
    public HitTestResultBehavior myCallback(HitTestResult result) //обработка при определении что фигура нажата
    {
        //при щелчке на визуальном элементе переключится между скошенным и нормальным состоянием
        if (result.VisualHit.GetType() == typeof(DrawingVisual))
        {
            if (((DrawingVisual)result.VisualHit).Transform == null)
            {
                ((DrawingVisual)result.VisualHit).Transform = new SkewTransform(5,5);
            }
        }
        else
        {
            ((DrawingVisual) result.VisualHit).Transform = null;
        }
        //прекращение поиска в визуальном делеве
        return HitTestResultBehavior.Stop;
    }
```
















