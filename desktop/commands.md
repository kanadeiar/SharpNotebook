# Команды

Команды являются неотъемлемой частью WPF, могут привязываться к элементам управления WPF для обработки пользовательских событий, подобных щелчку. Вместо обработчика события выполняется метод Execute() команды. Метод CanExecute() нужен для включения в работу элемента управления. Использование команд вместо обработчиков событий дает преимущество инкапсуляции кода приложения и автоматического включения и отключения элементов в окне приложения.

Интерфейс команды ICommand:

```csharp
public interface ICommand
{
    event EventHandler CanExecuteChanged;
    bool CanExecute(object parameter);
    void Execute(object parameter);
}
```

Возвращение методом CanExecute() значения true означает включенным состоянием элементов управления, если false - отключенные. Параметры, передаваемые обоим методам, поступают из пользовательского интерфейса. Событие CanExecuteChanged уведомляет пользовательский интерфейса в том, что результат метода CanExecute() изменился. Команда должна быть присоединена к диспетчеру команд (CommandManager). Каждый класс команды обязан присоедениться к диспетчеру команд.

Пример применения команды:

```csharp
<Button x:Name="buttonChangeColor" Content="Изменить цвет" Margin="5" Padding="4, 2"
        Command="{Binding RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type Window}}, Path=ChangeColorCommand}"
        CommandParameter="{Binding ElementName=comboBoxCars, Path=SelectedItem}"/>
class ChangeColorCommand : ICommand
{
    public bool CanExecute(object parameter) => (parameter as Inventory) != null;
    public void Execute(object parameter)
    {
        ((Inventory) parameter).Color = "Бурый";
    }
    public event EventHandler CanExecuteChanged
    {
        add => CommandManager.RequerySuggested += value;
        remove => CommandManager.RequerySuggested -= value;
    }
}
public partial class MainWindow : Window
{
    readonly IList<Inventory> cars = new ObservableCollection<Inventory>(); //
...
    //поддерживающее поле команды
    private ICommand _changeColorCommand = null;
    public ICommand ChangeColorCommand => _changeColorCommand ??= new ChangeColorCommand();
```

Для всех команд желательно создать базовый абстрактный класс команд, который будут наследовать все производные команды.

Пример базового класса команд и команды:

```csharp
<Button x:Name="buttonAddCar" Content="Добавить автомобиль" Margin="5" Padding="4, 2"
        Command="{Binding RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type Window}}, Path=AddCarCommand}"
        CommandParameter="{Binding ElementName=comboBoxCars, Path=ItemsSource}"/>
public abstract class CommandBase : ICommand
{
    public abstract bool CanExecute(object parameter);
    public abstract void Execute(object parameter);
    public event EventHandler CanExecuteChanged
    {
        add => CommandManager.RequerySuggested += value;
        remove => CommandManager.RequerySuggested -= value;
    }
}
class AddCarCommand : CommandBase
{
    public override bool CanExecute(object parameter)
    {
        return parameter is ObservableCollection<Inventory>;
    }
    public override void Execute(object parameter)
    {
        if (parameter is ObservableCollection<Inventory> cars)
        {
            var maxCount = cars.Max(x => x.CarId);
            cars.Add(new Inventory
            {
                CarId = ++maxCount,
                Color = "Желтый",
                Make = "ГАЗ",
                PetName = "Телега",
            });
        }
    }
}
public partial class MainWindow : Window
{
...
    //поддерживающее поле команды
    private ICommand _addCarCommand = null;
    public ICommand AddCarCommand => _addCarCommand ??= new AddCarCommand();
```

В качестве команд можно использовать класс RelayCommand. И использовать RelayCommand<T>, когда требуется параметр.
    
Пример класса RelayCommand и его примененения:

```csharp
public class RelayCommand : CommandBase
{
    private readonly Action execute;
    private readonly Func<bool> canExecute;
    public RelayCommand() {}
    public RelayCommand(Action execute) : this(execute, null) {}
    public RelayCommand(Action execute, Func<bool> canExecute)
    {
        this.execute = execute ?? throw new ArgumentNullException(nameof(execute));
        this.canExecute = canExecute;
    }
    public override bool CanExecute(object parameter)
    {
        return this.canExecute == null || this.canExecute();
    }
    public override void Execute(object parameter)
    {
        this.execute();
    }
}
public class RelayCommand<T> : RelayCommand
{
    private readonly Action<T> execute;
    private readonly Func<T, bool> canExecute;
    public RelayCommand() {}
    public RelayCommand(Action<T> execute) : this(execute, null) {}
    public RelayCommand(Action<T> execute, Func<T, bool> canExecute)
    {
        this.execute = execute ?? throw new ArgumentNullException(nameof(execute));
        this.canExecute = canExecute;
    }
    public override bool CanExecute(object parameter)
    {
        return this.canExecute == null || this.canExecute((T) parameter);
    }
    public override void Execute(object parameter)
    {
        this.execute((T) parameter);
    }
}
<Button x:Name="buttonDeleteCar" Content="Удалить авто" Margin="5" Padding="4, 2"
    Command="{Binding RelativeSource={RelativeSource Mode=FindAncestor, AncestorType={x:Type Window}}, Path=DeleteCarCommand}"
    CommandParameter="{Binding ElementName=comboBoxCars, Path=SelectedItem}"/>
public partial class MainWindow : Window
...
    private RelayCommand<Inventory> deleteCarCommand = null;
    public RelayCommand<Inventory> DeleteCarCommand => deleteCarCommand ??= new RelayCommand<Inventory>(DeleteCar, CanDeleteCar);
    private bool CanDeleteCar(Inventory car) => car != null;
    private void DeleteCar(Inventory car)
    {
        this.cars.Remove(car);
    }
```












