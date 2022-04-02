# On DataBinding in WPF

*"In computer programming, data binding is a general technique that binds data sources from the provider and consumer together and synchronizes them."* [1]

## Simple data binding

[2]

Binding between two controls to show how data binding works: binding between a TextBox and a TextBlock.

```xml
<TextBox Name="Source" />
<TextBlock Text="{Binding ElementName=Source, Path=Text}" />
```

In this example, the TextBlock's Text is bound to Source's Text property. The binding statement can be read as `TextBlock.Text = Source.Text` where `Source` is the name of the TextBox.

## The DataContext

[3]

The DataContext is the source object, an instance that provides data. In a very simple example, the `Window` can be bound to itself.

```c#
// In the constructor of the Window:
DataContext = this;
```

Then, the TextBox can be bound to any property of the `Window` the same way.

```xml
<TextBox Text="{Binding Title}" />
```

This can be read as `TextBox.Text = DataContext.Title` where the `DataContext` is `this`, and `this` is the current `Window`.

The DataContext cascades down, so the TextBox reaches up to the Window for its DataContext (the nearest parent with a DataContext).

## Observing

[4]

Binding to lists is done using the `ItemsSource` property on list-type controls. In this example, we have a list of users in the Window, and the Window is bound to itself.

```c#
// A property must be public and have both getters and setters.
List<User> users { get; set; }
```

```xml
<ListBox DisplayMemberPath="Name" ItemsSource="{Binding users}"></ListBox>
```

This binding reads `ListBox.Items = DataContext.users`. `DisplayMemberPath` means to use the `Name` property of the items of `users`: `User.Name`.

But the UI won't update if we add, change, or remove items in the `users` list. To observe changes in a collection, `ObservableCollection<T>` must be used. To observe changes in a type, the type must implement `INotifyPropertyChanged`.

```c#
// Won't work: changes in a List<T> won't show in bound UI.
public List<User> users { get; set; }

// Will work: ObservableCollection<T> will notify bound UI to update.
public ObservableCollection<User> users { get; set; }
```

The `INotifyPropertyChanged` interface raises an event that contains the name of the property that's changed. There are several ways of implementing this behavior, but this is possibly the most portable way.

```c#
// Previously, the object.
public class User
{
    public string Name { get; set; }
}

// ------------------------------------------------------

// With implementing INotifyPropertyChanged.
public class User: INotifyPropertyChanged
{
    // 1. A simple property is expanded into a full property.
    private string _name;
    public string Name
    {
        get { return _name; }
        set
        {
            // 2. The setter raises the event if it is a new value.
            it (_name != value)
            {
                _name = value;
                OnPropertyChanged();
            }
        }
    }

    // 3. Declaring the event that's raised.
    public event PropertyChangedEventHandler PropertyChanged;

    // 4. Implementing the event body. CallerMemberName gets the calling property name during compile time.
    public void OnPropertyChanged([CallerMemberName] string propertyName = "")
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

## Main-Detail view

[5]

In this example, a ListBox displays a list of objects. Clicking on one displays further information in a ContentControl.

```xml
<ListBox ItemsSource="{Binding People}" IsSynchronizedWithCurrentItem="True"/>
```

Here, we bind the ListBox's ItemsSource property to the People collection in the DataContext. An important property here is `IsSynchronizedWithCurrentItem`, which will ensure that the following ContentControl's data matches the selection in the ListBox.

```xml
<ContentControl Content="{Binding People}">
    <ContentControl.ContentTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Path=HomeTown}"/>
        </DataTemplate>
    </ContentControl.ContentTemplate>
</ContentControl>
```

Here, the binding is done the exact same way to the ListBox -- binding the ContentControl's Content property to the People collection in the DataContext. But in the template, we can bind to a property of an item in the collection: for example the HomeTown property in Person (as in: `let item: Person in People => item.HomeTown`).

## Sources

[1]: https://en.wikipedia.org/wiki/Data_binding

[2]: https://wpf-tutorial.com/data-binding/hello-bound-world/

[3]: https://wpf-tutorial.com/data-binding/using-the-datacontext/

[4]: https://wpf-tutorial.com/data-binding/responding-to-changes/

[5]: https://docs.microsoft.com/en-us/dotnet/desktop/wpf/data/how-to-bind-to-a-collection-and-display-information-based-on-selection?view=netframeworkdesktop-4.8
