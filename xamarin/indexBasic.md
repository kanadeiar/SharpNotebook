# Базовый Xamarin.Android

## Проект

Connected Services - подключение различных облачных сервисов.

Properties - описание всех требований приложения.

Ссылки - зависимости проекта.

Assets - файлы, необходимые в ходе работы приложения.

Resources - ресурсы программы.

MainActivity.cs - описание класса главной активности.

Resources/drawable - размещение графических ресурсов. Resource.Drawable.

Resources/mipmap - файлы значков с различной плотностью. Resource.Mipmap.

Resources/layout - файлы визуального дизайнера фрагментов, деятельностей. Resource.Layout.

Resources/values - файлы хранения простых значений - строк (Resource.String), чисел, цветов(Resource.Color).

Resources/Resource.designer.cs - файл определения класса Resource.

Resources/drawable-...dpi - графические ресурсы приложения для различных видов плотности экрана.

## Манифест

Файл манифеста приложения AndroidManifest.xml определяет требования приложения.

package - уникальное наименование пакета приложения

versionCode - внутренний номер версии, для сравнения версий приложений, используется магазином приложения для обновления приложений

versionName - название версии для пользователя, показывается пользователю

minSdkVersion - минимально возможный уровень API, требуемый для работы приложения

maxSdkVersion - самая поздняя версия API, поддерживаемое приложением

targetSdkVersion - целевая платформа, на которую разрабатывается приложение

android:allowBackup - участие приложения в функциях резервного копирования

icon - иконка приложения

label - отображаемая пользователю метка приложения

roundIcon - круглые значки для приложения

theme - тема по умолчанию для всех активностей приложения

## Базовое приложение

Проект -> Xamarin.Android.

Разметка экрана:
```csharp
<?xml version="1.0" ancoding="utf-8"?>
<RelativeLayout ...
    android:gravity="center">
    <LinearLayout
        android:id="@+id/lineatLayout1"
        android:orientation="vertical"
        android:minWidth="25px"
        android:minHeight="25px"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/textView1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Причет, мир!"
            android:gravity="center"/>
        <Button
            android:id="@+id/burron1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Нажми меня"/>
    </LinearLayout>
</RelativeLayout>
```
Код активности:
```csharp
//портретная ориентация
[Activity(Label = "@string/app_name", Theme = "@style/AppTheme", MainLauncher = true, ScreenOrientation = ScreenOrientation.Portrait)]
public class MainActivity : AppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
        SetContentView(Resource.Layout.activity_main);
        Debug.WriteLine("Выполнено OnCreate");
        //показать приложение при залоченом экране, включить экран принудительно
        Window.AddFlags(WindowManagerFlags.KeepScreenOn | WindowManagerFlags.ShowWhenLocked | WindowManagerFlags.TurnScreenOn);
 
        var buttonOk = FindViewById<Button>(Resource.Id.button1);
        buttonOk.Click += delegate
        {
            buttonOk.Text = "Закрыть?";
            var alert = new Android.App.AlertDialog.Builder(this);
            alert.SetTitle("Внимание");
            alert.SetMessage("Действительно выйти из приложения?");
            alert.SetCancelable(false);
            alert.SetPositiveButton("Да", delegate 
            { 
                Process.KillProcess(Process.MyPid()); 
            });
            alert.SetNegativeButton("Нет", delegate 
            { 
                Toast.MakeText(this, GetString(Resource.String.cancel_text), ToastLength.Short).Show(); 
            });
            RunOnUiThread(() => 
            {
                alert.Show();
            });
            
            var color = Color.Orange;
            Window.SetStatusBarColor(color);
        };  
    }
    protected override void OnDestroy()
    {
        base.OnDestroy();
        Debug.WriteLine(_tag, "Выполнено OnDestroy");
    }
}
```
Ресурсы в папке "values_ru" в файле "strings.xml" - локализация приложения:
```csharp
<resources>
  <string name="app_name">Учебное приложение 1</string>
  <string name="action_settings">Настройки</string>
  <string name="cancel_text">Отменено</string>
  <string name="hello_text">Привет, мир!</string>
  <string name="ok_text">Лады</string>
</resources>
```

