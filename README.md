## xam-forms-sqlitenet-async

A Xamarin Forms example using SQLite.NET-PCL with Async

I thought I'd whip up a quick example of how to use SQLite.NET with async in a Xamarin Forms PCL project,
with proper locking so you won't end up with deadlocks.

If there's anything wrong or missing, please send through a pull request :)

## โครงสร้างโปรเจค

```
└── SqliteAsyncExample
    ├── Droid
    │   ├── MainActivity.cs
    │   ├── Properties
    │   │   └── AssemblyInfo.cs
    │   └── SQLite_Android.cs
    ├── SqliteAsyncExample
    │   ├── Helpers
    │   │   └── AsyncSemaphore.cs
    │   ├── ISQLite.cs
    │   ├── Models
    │   │   └── Movie.cs
    │   ├── Properties
    │   │   └── AssemblyInfo.cs
    │   ├── Services
    │   │   ├── IMoviesService.cs
    │   │   └── MoviesService.cs
    │   ├── SqliteAsyncExample.cs
    │   ├── ViewModels
    │   │   └── MoviesViewModel.cs
    │   └── Views
    │       ├── MoviesPage.xaml
    │       └── MoviesPage.xaml.cs
    └── iOS
        ├── AppDelegate.cs
        ├── Main.cs
        └── SQLite_iOS.cs
```

## Packages

- SQLite.Net-PCL
- REfractored.MvvmHelpers

## Api ที่เกี่ยวข้อง

- DependencyService - https://developer.xamarin.com/guides/xamarin-forms/dependency-service
- BaseViewModel - https://github.com/jamesmontemagno/mvvm-helpers
- AsyncSemaphore - http://blogs.msdn.com/b/pfxteam/archive/2012/02/12/10266983.aspx
- AyncLock - http://blogs.msdn.com/b/pfxteam/archive/2012/02/12/10266988.aspx

## ใช้ DepedencyService สำหรับจัดการกับ Depedency

### Register dependency ผ่านแอททริบิวท์ Dependency

```csharp
[assembly: Dependency(typeof(SQLite_iOS))]
namespace SqliteAsyncExample.iOS
{
    public class SQLite_iOS : ISQLite
    { ... }
}
```

### ดึง Depedency ผ่าน DependencyService.Get

```csharp
var db = DependencyService.Get<ISQLite>().GetConnection();
db.CreateTable<Movie>();
```

## การ Insert และ Query

```csharp
public class MoviesService : IMoviesService {
     private static readonly AsyncLock Locker = new AsyncLock();

     private SQLiteAsyncConnection Database { get; } = DependencyService.Get<ISQLite>().GetAsyncConnection();

     public async Task AddMovies(IList<Movie> movies) {
         using (await Locker.LockAsync()) {
             await Database.InsertAllAsync(movies);
         }
     }

     public async Task<IList<Movie>> GetMovies() {
         using (await Locker.LockAsync()) {
             return await Database.Table<Movie>().Where(x => x.Id > 0).ToListAsync();
         }
     }
 }
```