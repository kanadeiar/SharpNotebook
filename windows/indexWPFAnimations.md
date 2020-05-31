# WPF Анимация

Инфраструктура WPF позволяет строить анимации целиком в разметке XAML, либо целиком в C#, либо их комбинацией. 

Каждый класс, именованный вида ТипДанныхAnimation (ByteAnimation, ColorAnimation, DoubleAnimation, Int32Animation) позволяет работать с анимацией линейной интерполяцией, обеспечивает плавное изменение значения во времени от начального к конечному.

Каждый класс, именованный вида ТипДанныхAnimationUsingKeyFrames (StringAnimationUsingKeyFrames, PointAnimationUsingKeyFrames, PointAnimationUsingKeyFrames) предоставляют анимацию ключевыми кадрами, позволяющей проходить в цикле по набору определенных значений за указанный период времени.

Каждый класс, именованный ТипДанныхAnimationUsingPath (DoubleAnimationUsingPath, PointAnimationUsingPath) предоставляют анимацию на основе пути, позволяющие перемещать объекты по заданному пути.

Классы Animation могут подключаться к любому свойству зависимости заданного объекта. В них определены следующие ключевые свойства для выполнения анимации:

Свойство To - предостапляет конечное значение анимации,

Свойство From - предоставляет начальное значение анимации,

Свойство By - общая величина, на которую анимация изменяет начальное значение.

Классы Animation наследуются от общего базового класса System.Windows.Media.Animation.Timeline, предоставляющий дополнительные свойства:
```csharp
AccelerationRatio, DeccelerationRatio, SpeedRatio   Свойства применяются для управления общим темпом анимационной последовательности
AutoReverse                 Получает или устанавливает значение что должна ли временная шкала воспроизводиться в обратном направлении после завершения итерации вперед, по умолчанию - false
BeginTime                   Получает или устанавливает время запуска временной шкалы, по умолчанию - 0
Duration                    Установка продолжительности воспроизведенения временной шкалы  
FileBehavior, RepeatBehavior    Управление тем, что должно произойти после завершения временной шкалы
```

## Анимация в C#

Пример использования анимации в коде C#:
```csharp
<StackPanel Orientation="Horizontal">
    <Button x:Name="buttonSpinner" Height="50" Width="100" Content="Спиннер" Margin="30" MouseEnter="ButtonSpinner_OnMouseEnter" Click="ButtonSpinner_OnClick"/>
</StackPanel>
private bool isSpinning = false; //булевская переменная чтобы анимация не началась сначало
private void ButtonSpinner_OnMouseEnter(object sender, MouseEventArgs e)
{
    if (!isSpinning)
    {
        isSpinning = true;
        var doubleAnim = new DoubleAnimation();
        doubleAnim.Completed += (o, args) => isSpinning = false;
        doubleAnim.From = 0;
        doubleAnim.To = 360;
        doubleAnim.Duration = new Duration(TimeSpan.FromSeconds(10)); //время на анимацию - 10 секунд
        var rotate = new RotateTransform();
        buttonSpinner.RenderTransform = rotate;
        rotate.BeginAnimation(RotateTransform.AngleProperty, doubleAnim); //анимация вращения
    }
}
private void ButtonSpinner_OnClick(object sender, RoutedEventArgs e)
{
    var doubleAnim = new DoubleAnimation
    {
        From = 1.0,
        To = 0.0,
    };
    //doubleAnim.RepeatBehavior = RepeatBehavior.Forever; //бесконечно повторять
    doubleAnim.RepeatBehavior = new RepeatBehavior(3); //повторить три раза
    //doubleAnim.RepeatBehavior = new RepeatBehavior(TimeSpan.FromSeconds(30)); //повторять 30 секунд
    doubleAnim.AutoReverse = true; //после завершения - запустить в обратном порядке
    buttonSpinner.BeginAnimation(Button.OpacityProperty, doubleAnim); //становление кнопки невидимой
}
```

## Анимация в XAML

Элементы Animation должны быть внутри элемента Storyboard, отображающий объект анимации на заданное свойство родительского объекта через свойство TargetProretry. Элемент Stroryboard должен всегда находится внутри BeginStroryboard.

Должно быть указано действие какого-то вида, которое приведет к запуску анимации, один из способов - это триггеры. Триггер - способ реагирования на событие в разметке XAML без необходимости написания процедурного кода, способ получить уведомление о том, что некоторое событие произошло.

После получение уведомления о появлении события можно запустить раскадровку.

Пример использования анимации в разметке XAML:
```csharp
<Label Content="Анимация" Margin="30">
    <Label.Triggers>
        <EventTrigger RoutedEvent="Label.Loaded">
            <EventTrigger.Actions>
                <BeginStoryboard>
                    <Storyboard TargetProperty="FontSize">
                        <DoubleAnimation From="12" To="100" Duration="0:0:5" RepeatBehavior="Forever" AutoReverse="True"/>
                    </Storyboard>
                </BeginStoryboard>
            </EventTrigger.Actions>
        </EventTrigger>
    </Label.Triggers>
</Label>
```
Анимация ключевыми кадрами в отличие от анимации линейной интерполяции, позволяет создавать коллекции специальных значений, которые должны достигатся в определенные моменты времени.

Пример использования анимации ключевыми кадрами в разметке XAML:
```csharp
<Button Name="myButton" Height="40" Width="150" FontSize="18" FontFamily="Verdana">
    <Button.Triggers>
        <EventTrigger RoutedEvent="Button.Loaded">
            <BeginStoryboard>
                <Storyboard>
                    <StringAnimationUsingKeyFrames RepeatBehavior="Forever" Storyboard.TargetProperty="Content" Duration="0:0:5">
                        <DiscreteStringKeyFrame Value="" KeyTime="0:0:0"/>
                        <DiscreteStringKeyFrame Value="O" KeyTime="0:0:1"/>
                        <DiscreteStringKeyFrame Value="OK" KeyTime="0:0:2"/>
                        <DiscreteStringKeyFrame Value="OK!" KeyTime="0:0:3"/>
                    </StringAnimationUsingKeyFrames>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```













