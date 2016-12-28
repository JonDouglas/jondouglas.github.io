---
layout: post
title: Xamarin.Android - Entity Framework
---

## Preface

Entity Framework has been one of my favorite projects for quite some time. If you didn't know, Entity Framework split off from the 6.X version to a new re-written 7.X (Core) version, in which the goal is to keep the ORM lightweight and very extensible.

Now the challenge for me has always been:

> I want to use Entity Framework in my Xamarin projects.

This hasn't really been possible for the last 3 years. In fact, I've attempted it a few years back and gave up because the tooling was simply just not there. However this is my personal redemption at getting this to work.

## Tools

Let's talk about the tooling needed to make this possible. Consider this more of a high level approach:

- SQLite Provider- We will need a place to store our data. Seeing that SQLite is a perfect fit for mobile, we will use Entity Framework's SQLite package: 

[https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/)

- Entity Framework Core - We will need an ORM to manage our CRUD operations and database migrations: 

[https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/)

Great! Now we're off to the races.

## Getting Started

### Creating the NetStandard library

The first thing we'll do is create a `netstandard 1.4` library. To do this, create a PCL project:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/413bf636-cd08-4ee3-816e-e2ea288908e6/12.28.2016-12.17.png)

Since this is not a `netstandard` library quite yet, let's go ahead and convert that by going to the project's `Properties`

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/e9035e1d-f3c8-4588-a73e-b1ed05514ae1/12.28.2016-12.19.png)

Once we have that set, we want to ensure we have the minimum requirements for the two NuGet packages we linked above:

[https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/)

[https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/)

By the looks of things, the dependency is `netstandard 1.3`. So let's make sure our `netstandard` project is targeting that version:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/934c42ef-20fe-416e-b9db-08233251362c/12.28.2016-12.21.png)

### Creating the Xamarin.Android project

Nothing too special here, we're just going to create a `File -> New Single=View App (Android) Project`:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/ae2f7ace-e0d2-4a64-8958-33953eac58a5/12.28.2016-12.22.png)

### Adding the NuGet packages

This step is typically easiest to do on the project level rather than the solution level as NuGet is sometimes not that friendly with different project structures(`.csproj` vs `project.json`)

### Adding the NuGet packages to the netstandard library

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/37444b40-49a7-4552-b9bb-f380a61b8627/12.28.2016-12.27.png)

### Adding the NuGet packages to the Xamarin.Android project

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/352d4d5c-d933-4a10-bff0-065885ef5cdb/12.28.2016-12.44.png)

## Defining the DbContext

If you've used Entity Framework before, you will be very familiar with how we define a `DbContext` and our underlying `Models` that define our database schema.

Let's start with a simple data model that we will call `Cat.cs`:

```
    public class Cat
    {
        [Key]
        public int CatId { get; set; }
        public string Name { get; set; }
        public int MeowsPerSecond { get; set; }
    }
```

Let's now make sure that this is apart of our `DbContext` by defining a new context we'll call `CatContext.cs`. You may notice that we have a `string DatabasePath`. We will use this later when we need to tell Entity Framework where to store our Database on disk:

```
    public class CatContext : DbContext
    {
        public DbSet<Cat> Cats { get; set; }

        private string DatabasePath { get; set; }

        public CatContext()
        {

        }

        public CatContext(string databasePath)
        {
            DatabasePath = databasePath;
        }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlite($"Filename={DatabasePath}");
        }
    }
```

## Implementing the Context

First, we need to make sure our Xamarin.Android project is referencing our `netstandard` library.

http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/e16ae9da-00ba-45f8-a912-6fb4a442586c/12.28.2016-13.01.png

Now that we have that, let's implement our `MainActivity.cs` with some Entity Framework code!

```
[Activity(Label = "EntityFrameworkWithXamarin.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    public class MainActivity : Activity
    {
        int count = 1;

        protected async override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);

            // Set our view from the "main" layout resource
            SetContentView(Resource.Layout.Main);

            // Get our button from the layout resource,
            // and attach an event to it
            Button button = FindViewById<Button>(Resource.Id.MyButton);

            button.Click += delegate { button.Text = string.Format("{0} clicks!", count++); };

            TextView textView = FindViewById<TextView>(Resource.Id.TextView1);

            var dbFolder = System.Environment.GetFolderPath(System.Environment.SpecialFolder.Personal);
            var fileName = "Cats.db";
            var dbFullPath = Path.Combine(dbFolder, fileName);
            try
            {
                using (var db = new CatContext(dbFullPath))
                {
                    await db.Database.MigrateAsync(); //We need to ensure the latest Migration was added. This is different than EnsureDatabaseCreated.

                    Cat catGary = new Cat() { CatId = 1, Name = "Gary", MeowsPerSecond = 5 };
                    Cat catJack = new Cat() { CatId = 2, Name = "Jack", MeowsPerSecond = 11 };
                    Cat catLuna = new Cat() { CatId = 3, Name = "Luna", MeowsPerSecond = 3 };

                    List<Cat> catsInTheHat = new List<Cat>() { catGary, catJack, catLuna };

                    if(await db.Cats.CountAsync() < 3)
                    {
                        await db.Cats.AddRangeAsync(catsInTheHat);
                        await db.SaveChangesAsync();
                    }

                    var catsInTheBag = await db.Cats.ToListAsync();

                    foreach(var cat in catsInTheBag)
                    {
                        textView.Text += $"{cat.CatId} - {cat.Name} - {cat.MeowsPerSecond}" + System.Environment.NewLine;
                    }
                }

            }
            catch (Exception ex)
            {
                System.Diagnostics.Debug.WriteLine(ex.ToString());
            }
        }
    }
```

Let's try to run this code.

```
Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 1: 'no such table: Cats'.
```

It looks like we're missing a core Entity Framework feature, and that's `Migrations` to create our database schema.

Well shucks...this is awkward. We don't have a great way to generate Entity Framework `Migrations` from within a Xamarin.Android project or the `netstandard` library. Let's work with a quick workaround by creating a new `netcore` Console Application so we can generate `Migrations`.

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/329e9a3b-6065-4a90-b499-e6d20ff70616/12.28.2016-13.56.png)

We need to add the `Entity Framework Tools`, `Entity Framework Core Design`, and `Entity Framework Core` to this project so we can use the command line to generate our `Migrations`.

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/947ef73a-3b84-46b0-a56e-0e93d5533f3f/12.28.2016-13.57.png)

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/60d1647f-3243-43df-a692-e7cdd0cac4cc/12.28.2016-13.59.png)

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/1f368ad3-0145-4bee-8ddd-3638661586b9/12.28.2016-14.27.png)

We now need to move over our `Cat.cs` and `CatContext.cs` to ensure there's a `DbContext` it can generate `Migrations` for. Your Console App should now look like this:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/8ae84f5a-6ffb-4e64-9df1-98d69ac307bd/12.28.2016-14.02.png)

Now we can generate a schema for our context. Let's use the new `dotnet` tooling to do this. Open up a new console in our current Console App directory:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/27b633be-12fd-40e6-969b-0d9cb9e3aec0/12.28.2016-14.03.png)

Now we need to generate an initial `Migration`.

`dotnet ef migrations add Initial`

This should generate a migration:

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/fd90bc04-3e22-49a3-b9e7-0d37aa79a590/12.28.2016-14.11.png)

Now we need to take the initial migrations generated in the `Migrations` folder of our project and simply move them over to our `netstandard` library.

**Note:** You can simply change the namespaces of these two generated files to the name of your `netstandard` namespace.

Let's try running the Xamarin.Android project again and see if we run into any other exceptions

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/3ec8d330-8114-47ed-b5a4-8e48a69bf70e/12.28.2016-14.16.png)

It looks like it worked! Our simple attempt at adding `Cat` models and retrieving them works!

If we wanted to take a closer look at the `SQLite` file that gets generated, use an emulator and open up ADM(Android Device Monitor):

Taking a closer look into the `data/data/files` folder, we will see our `Cats.db` that we created.

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/145c9d28-0320-41e1-8ea4-dfb2b564ca73/12.28.2016-14.19.png)

You can now take that file and open it in any `SQLite` explorer.

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/ae6bd4bf-2e4e-442d-a12e-3b17f61282ef/12.28.2016-14.20.png)

**Note:** I personally use DB Browser for SQLite ([http://sqlitebrowser.org/](http://sqlitebrowser.org/))

## Conclusion

**Source Code:** [https://github.com/JonDouglas/EntityFrameworkWithXamarin](https://github.com/JonDouglas/EntityFrameworkWithXamarin)

It's been a three year battle with you Entity Framework. However I have to give my thanks to everyone involved in getting Entity Framework to the state it currently is. It's been so much fun seeing how these projects turn out after a long period of development.