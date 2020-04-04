# Шаблон графического приложения

1. Создать пустой проект С#.

2. Добавиь ссылки на "System.Drawing" и "System.Windows.Forms".

3. Добавить файл "Program.cs":
```csharp
using System;
using System.Windows.Forms;

namespace Game
{
    class Program
    {
        static void Main(string[] args)
        {
            Form form = new Form();
            form.Width = 800;
            form.Height = 600;
            form.Text = "Game \"Asteroids\"";
            Game.Init(form);
            form.Show(); //показ формы
            Game.Draw();
            Application.Run(form); //запуск выполения приложения
        }
    }
}
```

4. В свойствах приложения установить - "Приложение Windows".

5. Добавить файл "Game.cs":
```csharp
using System;
using System.Windows.Forms;
using System.Drawing;

namespace Game
{
    static class Game
    {
        private static BufferedGraphicsContext context;
        public static BufferedGraphics Buffer;
        public static int Width {get;set;}
        public static int Height {get;set;}
        static Game()
        {
        }
        public static void Init(Form form)
        {
            Graphics g; //графическое устройство для вывода графики
            context = BufferedGraphicsManager.Current; //буфер графического контекста для приложения
            g = form.CreateGraphics();
            Width = form.ClientSize.Width;
            Height = form.ClientSize.Height;
            Buffer = context.Allocate(g, new Rectangle(0,0,Width,Height)); //связь буфера в памяти с графическим объектом в целях рисования в буфере
        }
        public static void Draw()
        {
            Buffer.Graphics.Clear(Color.Black); //фон
            Buffer.Graphics.DrawRectangle(Pens.White, new Rectangle(100,100,200,200)); //квадрат
            Buffer.Graphics.FillEllipse(Brushes.Wheat, new Rectangle(100,100,200,200)); //круг
            Buffer.Render(); //перерисование
        }

    }
}
```
6. Добавление базового объекта-кружочка.

Новый файл "BaseObject.cs":
```csharp
using System;
using System.Drawing;

namespace Game
{
    class BaseObject
    {
        protected Point Pos;
        protected Point Dir;
        protected Size Size;
        public BaseObject(Point pos, Point dir, Size size)
        {
            Pos = pos;
            Dir = dir;
            Size = size;
        }
        public virtual void Draw(Graphics g)
        {
            g.DrawEllipse(Pens.White, Pos.X, Pos.Y, Size.Width, Size.Height);
        }
        public virtual void Update()
        {
            Pos.X += Dir.X;
            Pos.Y += Dir.Y;
            if (Pos.X < 0) Dir.X = -Dir.X;
            if (Pos.X+Size.Width > Game.Width) Dir.X = -Dir.X;
            if (Pos.Y < 0) Dir.Y = -Dir.Y;
            if (Pos.Y+Size.Height > Game.Height) Dir.Y = -Dir.Y;
        }
    }
}
```
Добавить обработчик таймера и таймер:
```csharp
//в Init() добавление метода загрузки новых элементов
    Load();
    Timer timer = new Timer {Interval = 10};
    timer.Tick += Timer_Tick;
    timer.Start();
private static void Timer_Tick(object sender, EventArgs e)
{
    Update();
    Draw();
}
public static void Draw()
{
    Buffer.Graphics.Clear(Color.Black); //фон
    foreach (BaseObject obj in objs)
        obj.Draw(Buffer.Graphics);
    Buffer.Render(); //перерисование
}
public static void Update()
{
    foreach (BaseObject obj in objs)
        obj.Update();
}
public static BaseObject[] objs;
public static void Load()
{
    objs = new BaseObject[30];
    for (int i=0; i<objs.Length; i++)
        objs[i] = new BaseObject(new Point(600, i * 20), new Point(15-i, 15-i), new Size(20,20));
}
```

7. Добавление нового дополнительного отображаемого объекта.

Новый файл "Star.cs":
```csharp
using System;
using System.Drawing;

namespace Game
{
    class Star : BaseObject
    {
        public Star(Point pos, Point dir, Size size) : base(pos, dir, size)
        {
        }
        public override void Draw(Graphics g)
        {
            g.DrawLine(Pens.White, Pos.X, Pos.Y, Pos.X+Size.Width, Pos.Y+Size.Height);
            g.DrawLine(Pens.White, Pos.X+Size.Width, Pos.Y, Pos.X, Pos.Y+Size.Height);
        }
        public override void Update()
        {
            Pos.X += Dir.X;
            if (Pos.X < 0)
                Pos.X = Game.Width + Size.Width;
        }
    }
}
```
Переделать метод загрузки новых объектов:
```csharp
public static void Load()
{
    objs = new BaseObject[20];
    for (int i = 0; i < objs.Length/2; i++)
        objs[i] = new BaseObject(new Point(600, i*20), new Point(-i,-i), new Size(10, 10));
    for (int i = objs.Length/2; i < objs.Length; i++)
        objs[i] = new Star(new Point(600, i*20), new Point(-i,0), new Size(10, 10));
}
```

