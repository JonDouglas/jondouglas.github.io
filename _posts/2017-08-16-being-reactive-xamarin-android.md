---
layout: post
title: Being Reactive with Xamarin.Android
---

# Preface

This is a continuation post of <http://www.jon-douglas.com/2017/08/01/being-more-reactive/> in which we will explore how we can apply **Reactive Extensions(RX)** to our Xamarin.Android project.

We will visually show how **Reactive Extensions(RX)** and a touch of **Reactive UI(RxUI)** to accomplish our previous example in Xamarin.Android

## Introducing ReactiveUI

[ReactiveUI](https://reactiveui.net/) is an advanced, composable, functional reactive model-view-viewmodel framework for all .NET platforms. The cool thing about ReactiveUI is that it can be used in small sections of your project, or it can be the main driving force behind your project. It's really up to you to decide.

ReactiveUI has documentation on each platform for best practices. For Xamarin.Android, we will follow a couple of best practices for our sample:

<https://reactiveui.net/docs/guidelines/platform/xamarin-android>

## Creating the Model

Let's define the models and helper classes for our BBQ:

```
 public class Hamburger
    {
        public Meat Meat { get; set; }
        public Lettuce Lettuce { get; set; }
        public Bun Bun { get; set; }
    }

    public class Meat
    {
        public bool Cooked { get; set; }
        public bool Rotten { get; set; }
    }

    public class Lettuce
    {
        public bool Fresh { get; set; } = true;
    }

    public class Bun
    {
        public bool Heated { get; set; }
    }

    public static class BBQHelpers
    {
        public static Meat Cook(Meat meat)
        {
            meat.Cooked = true;
            return meat;
        }

        public static Bun Heat(Bun bun)
        {
            bun.Heated = true;
            return bun;
        }
    }
```

## Creating the ViewModel

Let's create a `BurgerViewModel` that inherits `ReactiveObject`. We can then define our streams similar to how we did in the previous blog post:

```
public class BurgerViewModel : ReactiveObject
    {
        public IObservable<Bun> BunStream()
        {
            return Observable.Interval(TimeSpan.FromSeconds(5))
                .Select(_ => BBQHelpers.Heat(new Bun()))
                .Take(4)
                .Publish()
                .RefCount();
        }

        public IObservable<Meat> RawMeatStream()
        {
            return Observable.Interval(TimeSpan.FromSeconds(3))
                .Select(_ => new Meat())
                .Take(4)
                .Publish()
                .RefCount();
        }

        public IObservable<Meat> CookedMeatStream()
        {
            return RawMeatStream().Select(meat => BBQHelpers.Cook(meat))
                .Take(4)
                .Publish()
                .RefCount();
        }

        public IObservable<Lettuce> LettuceStream()
        {
            return Observable.Interval(TimeSpan.FromSeconds(2))
                .Select(_ => new Lettuce())
                .Take(4)
                .Publish()
                .RefCount();
        }

        public IObservable<Hamburger> HamburgerStream()
        {
            return Observable.Zip(CookedMeatStream(), BunStream(), LettuceStream(), (meat, bun, lettuce) => new Hamburger { Meat = meat, Bun = bun, Lettuce = lettuce })
                .Take(4)
                .Publish()
                .RefCount();
        }
    }
```

This ViewModel will serve as our various streams for our BBQ. This sample is only going to take 4 items each as our layout can only hold this amount visually without scrolling.

## Defining the Layout

We are going to define a very simple `LinearLayout` which includes 5 child `LinearLayouts` to show our BBQ streams:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:id="@+id/activity_bbq"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

  <LinearLayout
    android:id="@+id/raw_meat_layout"
    style="@style/StreamLayout" />

  <LinearLayout
    android:id="@+id/cooked_meat_layout"
    style="@style/StreamLayout" />

  <LinearLayout
    android:id="@+id/bun_layout"
    style="@style/StreamLayout" />

  <LinearLayout
    android:id="@+id/lettuce_layout"
    style="@style/StreamLayout" />

  <LinearLayout
    android:id="@+id/burger_layout"
    style="@style/StreamLayout" />
</LinearLayout>
```

## Creating the Activity

The activity will have a couple of special setup items. The first is that we need to inherit from a `ReactiveActivity<BurgerViewModel>`. This makes our definition the following:

`public class BurgersActivity : ReactiveActivity<BurgerViewModel>`

Now we need to wire up our controls. We can do this by the following definitions:

```
private LinearLayout bunLinearLayout;
private LinearLayout rawMeatLinearLayout;
private LinearLayout cookedMeatLinearLayout;
private LinearLayout lettuceLinearLayout;
private LinearLayout burgerLinearLayout;
```

We can then wire these up in the `OnCreate(Bundle bundle)` method:

```
bunLinearLayout = FindViewById<LinearLayout>(Resource.Id.bun_layout);
rawMeatLinearLayout = FindViewById<LinearLayout>(Resource.Id.raw_meat_layout);
cookedMeatLinearLayout = FindViewById<LinearLayout>(Resource.Id.cooked_meat_layout);
lettuceLinearLayout = FindViewById<LinearLayout>(Resource.Id.lettuce_layout);
burgerLinearLayout = FindViewById<LinearLayout>(Resource.Id.burger_layout);
```

Because we inherited from `ReactiveActivity<BurgerViewModel>`, we now need to ensure we are creating a new ViewModel in our `OnCreate(Bundle bundle)`:

`this.ViewModel = new BurgerViewModel();`

Our final definition of the `OnCreate(Bundle bundle)` should look like this:

```
protected override void OnCreate(Bundle bundle)
{
    base.OnCreate(bundle);

    SetContentView (Resource.Layout.activity_bbq);

    this.ViewModel = new BurgerViewModel();

    bunLinearLayout = FindViewById<LinearLayout>(Resource.Id.bun_layout);
    rawMeatLinearLayout = FindViewById<LinearLayout>(Resource.Id.raw_meat_layout);
    cookedMeatLinearLayout = FindViewById<LinearLayout>(Resource.Id.cooked_meat_layout);
    lettuceLinearLayout = FindViewById<LinearLayout>(Resource.Id.lettuce_layout);
    burgerLinearLayout = FindViewById<LinearLayout>(Resource.Id.burger_layout);
}
```
## Creating an extension method to add an image to the LinearLayout

To visually show our BBQ, we will want to create a helper method to resize a `Resource`, set the `ImageView`'s `Source`, and add it to the `LineaLayout`. Let add this in the `BurgersActivity.cs`:

```
private void AddImageToContainer(LinearLayout container, int imageSource)
{
    int width = Resources.GetDimensionPixelSize(Resource.Dimension.image_max_width);
    LinearLayout.LayoutParams viewParams = new LinearLayout.LayoutParams(width, ViewGroup.LayoutParams.WrapContent);
    ImageView imageView = new ImageView(this);
    imageView.SetImageResource(imageSource);
    container.AddView(imageView, viewParams);
}
```

## Creating extension methods for each BBQ task

Now we simply need a few helper methods to add a different image based on the BBQ task. Let's add this to the `BurgersActivity.cs`:

```
private void SetBun(Bun bun)
{
    AddImageToContainer(bunLinearLayout, Resource.Drawable.bun);
}

private void SetRawMeat(Meat meat)
{
    if (meat.Rotten)
    {
        AddImageToContainer(rawMeatLinearLayout, Resource.Drawable.rawMeatRotten);
    }
    else
    {
        AddImageToContainer(rawMeatLinearLayout, Resource.Drawable.rawMeat);
    }
}

private void SetCookedMeat(Meat meat)
{
    AddImageToContainer(cookedMeatLinearLayout, Resource.Drawable.cookedMeat);
}

private void SetLettuce(Lettuce lettuce)
{
    AddImageToContainer(lettuceLinearLayout, Resource.Drawable.lettuce);
}

private void SetHamburger(Hamburger burger)
{
    AddImageToContainer(burgerLinearLayout, Resource.Drawable.burger);
}
```

## Subscribing to our streams with `WhenActivated()`

Now the fun part, we can now subscribe to our ViewModel's streams. To do this, we will use the `WhenActivated()` in our `BurgersActivity` constructor. We will then be creating a `CompositeDisposable` to create a collection of `IDisposable` which are created from our subscriptions.

```
        public BurgersActivity()
        {
            this.WhenActivated(() =>
                {
                    var disposable = new CompositeDisposable();

                    disposable.Add(ViewModel.BunStream()
                        .SubscribeOn(Scheduler.Default)
                        .ObserveOn(RxApp.MainThreadScheduler)
                        .Subscribe(Observer.Create<Bun>(bun => SetBun(bun))));

                    disposable.Add(ViewModel.RawMeatStream()
                        .SubscribeOn(Scheduler.Default)
                        .ObserveOn(RxApp.MainThreadScheduler)
                        .Subscribe(Observer.Create<Meat>(meat => SetRawMeat(meat))));

                    disposable.Add(ViewModel.LettuceStream()
                        .SubscribeOn(Scheduler.Default)
                        .ObserveOn(RxApp.MainThreadScheduler)
                        .Subscribe(Observer.Create<Lettuce>(lettuce => SetLettuce(lettuce))));

                    disposable.Add(ViewModel.CookedMeatStream()
                        .SubscribeOn(Scheduler.Default)
                        .ObserveOn(RxApp.MainThreadScheduler)
                        .Subscribe(Observer.Create<Meat>(meat => SetCookedMeat(meat))));

                    disposable.Add(ViewModel.HamburgerStream()
                        .SubscribeOn(Scheduler.Default)
                        .ObserveOn(RxApp.MainThreadScheduler)
                        .Subscribe(Observer.Create<Hamburger>(hamburger => SetHamburger(hamburger))));

                    return disposable;
                }

            );
        }
```

You may notice that we have `SubscribeOn` and `ObserveOn` items defined. Simply put, `SubscribeOn` refers to the thread in which the actual call to subscribe happens, and `ObserveOn` refers to the thread in which the subscription is observed on.

We are then creating a new object and using our extension method to visually add an image to the respective `LinearLayout`.

## Summary

Let's see what we did in action:

![](http://i.imgur.com/yCJvRkb.gif)

We've only seen the tip of the iceberg of what Reactive Extensions and ReactiveUI can do for our applications. It is quite a powerful set of libraries that we can use to ease the complexity of our applications.

Source Code: <https://github.com/JonDouglas/BeingReactive>

## Xamarin.Android Book

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](http://eepurl.com/cz_fj1)