# WPF Визуализация

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
















