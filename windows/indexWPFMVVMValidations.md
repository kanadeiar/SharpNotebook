# WPF MVVM Валидация

Приложениям необходимо проверять пользовательский ввод и обеспечивать обратную связь с пользователем, если введенные им данные оказываются некорректными. Проверка достоверности происходит, когда привязка данных пытается обновить источник данных. Если правило проверки достоверности нарушается, то в игру вступает класс Validation.

Класс Validation применяется для проверки достоверности и предоставляет методы и свойства для отображения результата проверки. 

При обработке ошибок проверки используются три основных класса Validation:
```csharp
HasError            Присоединяемое свойство, указывающее что правило проверки достоверности где то в процессе было нарушено
Error               Коллекция всех активных объектов ValidationError
ErrorTemplate       Шаблон элемента управления, видимый и декорирующий связанный элемент, когда свойство HasError установлено в true
```
После внесения изменений индексатор вызывается на модели каждый раз при PropertyChanged. Если индексатор возвращает отличное от string.Empty, тогда в свойстве этого элемента присутствует ошибка, из-за чего каждый элемент управления, привязанный к свойству специфического экземпляра класса, считается содержащим ошибку. Свойство HasError устанавливается в true и показывается декоратор ErrorTemplate для элементов управления.

Интерфейс IDataErrorInfo дает механизм для добавления специальной проверки достоверности в классы моделей. Интерфейс добавляется в классы моделей, а код проверки помещается внутрь классов моделей.

Интерфейс:
```csharp
public interface IDataErrorlnfo
{
string this[string columnName] { get; }
string Error { get; }
}
```

Пример добавления в частичный класс интерфейса валидации IDataErrorInfo:
```csharp
...
</Grid.RowDefinitions>
<Label Grid.Column="0" Grid.Row="0" Content="Id" Margin="3"/>
<TextBox Grid.Column="1" Grid.Row="0" Margin="3" Text="{Binding Path=CarId, ValidatesOnDataErrors=True}"/>
<Label Grid.Column="0" Grid.Row="1" Content="Марка" Margin="3"/>
<TextBox Grid.Column="1" Grid.Row="1" Margin="3" Text="{Binding Path=Make, ValidatesOnDataErrors=True}"/>
...
public partial class Inventory : IDataErrorInfo
{
    private string error;
    //индексатор валидации
    public string this[string columnName]
    {
        get
        {
            switch (columnName)
            {
                case nameof(CarId):
                    break;
                case nameof(Make):
                    if (Make == "Чайка")
                    {
                        return "Слишком старая";
                    }
                    return CheckMakeAndColor();
                case nameof(Color):
                    return CheckMakeAndColor();
                case nameof(PetName):
                    break;
            }
            return string.Empty;
        }
    }
    private string CheckMakeAndColor()
    {
        if (Make == "Копейка" && Color == "Беж")
        {
            return $"{Make} не может быть цвета {Color}";
        }
        return string.Empty;
    }
    public string Error => error;
}
```
Интерфейс INotifyDataErrorInfo построен на основе IDataErrorInfo и дает дополнительные возможности для проверки достоверности.

Интерфейс:
```csharp
public interface INotifyDataErrorlnfo
{
bool HasErrors { get; }
event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
IEnumerable GetErrors(string propertyName);
}
```
Свойство HasError используется механизмом привязки для выяснения, есть ли какие либо ошибки в свойствах экземпляра. При реализации интерфейса INotifyDataErrorInfo требуется писать гораздо больше кода. При этом используются определенные механизмы этого интерфейса для проверки на предмет ошибок - блоки set для свойств. Это делает код труднее в чтении и сопровождении.

Пример валидации параметра с применением интерфейса INotifyDataErrorInfo для проверки достоверности (PropertyChanged.Fody не получится использовать):
```csharp
public partial class Inventory : INotifyPropertyChanged //наблюдаемая модель
{
...
    private string make;
    public string Make
    {
        get => make;
        set
        {
            if (value == make) return;
            make = value;
            //проверка достоверности
            if (Make == "Чайка") 
            {
                AddError(nameof(Make), "Чайка");
            }
            else
            {
                ClearErrors(nameof(Make));
            }
            //передача названия свойства значение которого изменилось
            OnPropertyChanged(nameof(Make));
        }
    }
...
public partial class Inventory : INotifyDataErrorInfo
{
    //хранение всех сведений о ошибках, сгруппированные по именам свойств
    private readonly Dictionary<string, List<string>> errors = new Dictionary<string, List<string>>();
    //возвращает любые ошибки в словаре, когда в параметре пустая сторока или null, если имя свойства - то ошибки этого свойства
    public IEnumerable GetErrors(string propertyName)
    {
        if (string.IsNullOrEmpty(propertyName))
        {
            return errors.Values;
        }
        return errors.ContainsKey(propertyName) ? errors[propertyName] : null;
    }
    //истина если в словаре присуствуют любые ошибки
    public bool HasErrors => errors.Count != 0;
    public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
    //вспомогательный метод инициирования события ErrorChanged
    private void OnErrorChanged(string propertyName)
    {
        ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(propertyName));
    }
    //добавление ошибок
    private void AddError(string propertyName, string error)
    {
        AddErrors(propertyName, new List<string>{error});
    }
    private void AddErrors(string propertyName, IList<string> errors)
    {
        var changed = false;
        if (!this.errors.ContainsKey(propertyName))
        {
            this.errors.Add(propertyName, new List<string>());
            changed = true;
        }
        foreach (var err in errors)
        {
            if (this.errors[propertyName].Contains(err)) continue;
            this.errors[propertyName].Add(err);
            changed = true;
        }
        if (changed)
            OnErrorChanged(propertyName);
    }
    //удаление ошибок
    protected void ClearErrors(string propertyName = "")
    {
        if (string.IsNullOrEmpty(propertyName))
            errors.Clear();
        else
            errors.Remove(propertyName);
        OnErrorChanged(propertyName);
    }
}
```
Пример валидации параметра с применением комбинирования интерфейсов IDataErrorInfo и INotifyDataErrorInfo для проверки достоверности (Теперь с PropertyChanged.Fody):
```csharp
public partial class Inventory : INotifyPropertyChanged
{
    public bool IsChanged { get; set; }
    public int CarId { get; set; }
    public string Make { get; set; }
    public string Color { get; set; }
    public string PetName { get; set; }
    public event PropertyChangedEventHandler PropertyChanged;
}
public partial class Inventory : IDataErrorInfo, INotifyDataErrorInfo
{
    //хранение всех сведений о ошибках, сгруппированные по именам свойств
    private readonly Dictionary<string, List<string>> errors = new Dictionary<string, List<string>>();
    //возвращает любые ошибки в словаре, когда в параметре пустая сторока или null, если имя свойства - то ошибки этого свойства
    public IEnumerable GetErrors(string propertyName)
    {
        if (string.IsNullOrEmpty(propertyName))
        {
            return errors.Values;
        }
        return errors.ContainsKey(propertyName) ? errors[propertyName] : null;
    }
    //истина если в словаре присуствуют любые ошибки
    public bool HasErrors => errors.Count != 0;
    public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
    //вспомогательный метод инициирования события ErrorChanged
    private void OnErrorChanged(string propertyName)
    {
        ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(propertyName));
    }
    //добавление ошибок
    private void AddError(string propertyName, string error)
    {
        AddErrors(propertyName, new List<string>{error});
    }
    private void AddErrors(string propertyName, IList<string> errors)
    {
        var changed = false;
        if (!this.errors.ContainsKey(propertyName))
        {
            this.errors.Add(propertyName, new List<string>());
            changed = true;
        }
        foreach (var err in errors)
        {
            if (this.errors[propertyName].Contains(err)) continue;
            this.errors[propertyName].Add(err);
            changed = true;
        }
        if (changed)
            OnErrorChanged(propertyName);
    }
    //удаление ошибок
    protected void ClearErrors(string propertyName = "")
    {
        if (string.IsNullOrEmpty(propertyName))
            errors.Clear();
        else
            errors.Remove(propertyName);
        OnErrorChanged(propertyName);
    }
    private string error;
    //индексатор валидации
    public string this[string columnName]
    {
        get
        {
            bool hasError = false;
            switch (columnName)
            {
                case nameof(CarId):
                    break;
                case nameof(Make):
                    hasError = CheckMakeAndColor();
                    if (Make == "Чайка")
                    {
                        AddError(nameof(Make), "Слишком старая");
                        hasError = true;
                    }
                    if (!hasError)
                    {
                        ClearErrors(nameof(Make));
                        ClearErrors(nameof(Color));
                    }
                    break;
                case nameof(Color):
                    hasError = CheckMakeAndColor();
                    if (!hasError)
                    {
                        ClearErrors(nameof(Make));
                        ClearErrors(nameof(Color));
                    }
                    break;
                case nameof(PetName):
                    break;
            }
            return string.Empty;
        }
    }
    private bool CheckMakeAndColor()
    {
        if (Make == "Копейка" && Color == "Беж")
        {
            AddError(nameof(Make), $"{Make} не может быть цвета {Color}");
            AddError(nameof(Color), $"{Make} не может быть цвета {Color}");
            return true;
        }
        return false;
    }
    public string Error => error;
}
```
Отображение ошибок класса Validation доступно различными способами.

Пример отображения найденных ошибок в списке в окне приложения:
```csharp
<ListBox Grid.Column="0" Grid.Row="6" Grid.ColumnSpan="2" ItemsSource="{Binding ElementName=DetailsGrid, Path=(Validation.Errors)}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <ListBox ItemsSource="{Binding Path=ErrorContent}"/>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

Возможно применение аннотаций данных для проверки достоверности в пользовательских интерфейсах инфтраструктуры WPF. Чтоб их использовать нужно добавить ссылку на сборку System.ComponentModel.DataAnnotation.dll.

Добавление атрибутов [Required] и [StringLength(50)] к свойствам класса-модели определяют правило проверки достоверности.

Пример добавления валидации через добавление аннотаций к свойствам класса и отображения предупреждений изменением стиля XAML:
```csharp
<Window.Resources>
    <Style TargetType="{x:Type TextBox}">
        <Style.Triggers>
            <Trigger Property="Validation.HasError" Value="True">
                <Setter Property="Background" Value="Pink"/>
                <Setter Property="Foreground" Value="Black"/>
                <Setter Property="ToolTip" 
                        Value="{Binding RelativeSource={RelativeSource Self},
                        Path=(Validation.Errors)[0].ErrorContent}"/>
            </Trigger>
        </Style.Triggers>
        <Setter Property="Validation.ErrorTemplate">
            <Setter.Value>
                <ControlTemplate>
                    <DockPanel LastChildFill="True">
                        <TextBlock Foreground="Red" FontSize="20" Text="!"
                                   ToolTip="{Binding ElementName=controlWithError,
                                   Path=AdornedElement.(Validation.Errors)[0].ErrorContent}"/>
                        <Border BorderBrush="Red" BorderThickness="1">
                            <AdornedElementPlaceholder Name="controlWithError"/>
                        </Border>
                    </DockPanel>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</Window.Resources>
...
//метод добавленный к другим методам, таким как GetErrors
protected string[] GetErrorsFromAnnotations<T>(string propertyName, T value)
{
    var results = new List<ValidationResult>();
    var vc = new ValidationContext(this, null, null) {MemberName = propertyName};
    var isValid = Validator.TryValidateProperty(value, vc, results);
    return (isValid) ? null : Array.ConvertAll(results.ToArray(), o => o.ErrorMessage);
}
...
protected void AddErrors(string propertyName, IList<string> errors)
{
    if (errors == null || errors?.Count == 0) 
        return;
...
public partial class Inventory : INotifyPropertyChanged
{
    public bool IsChanged { get; set; }
    [Required]
    public int CarId { get; set; }
    [Required][StringLength(50)]
    public string Make { get; set; }
    [Required][StringLength(50)]
    public string Color { get; set; }
    [StringLength(50)]
    public string PetName { get; set; }
...
public string this[string columnName]
{
    get
    {
        bool hasError = false;
        switch (columnName)
        {
            case nameof(CarId):
                AddErrors(nameof(CarId), GetErrorsFromAnnotations(nameof(CarId),CarId));
                break;
            case nameof(Make):
                hasError = CheckMakeAndColor();
                if (Make == "Чайка")
                {
                    AddError(nameof(Make), "Слишком старый автомобиль");
                    hasError = true;
                }
                if (!hasError)
                {
                    ClearErrors(nameof(Make));
                    ClearErrors(nameof(Color));
                }
                AddErrors(nameof(Make), GetErrorsFromAnnotations(nameof(Make), Make));
                break;
            case nameof(Color):
                hasError = CheckMakeAndColor();
                if (!hasError)
                {
                    ClearErrors(nameof(Make));
                    ClearErrors(nameof(Color));
                }
                AddErrors(nameof(Color), GetErrorsFromAnnotations(nameof(Color), Color));
                break;
            case nameof(PetName):
                AddErrors(nameof(PetName), GetErrorsFromAnnotations(nameof(PetName), PetName));
                break;
        }
        return string.Empty;
    }
}
```

