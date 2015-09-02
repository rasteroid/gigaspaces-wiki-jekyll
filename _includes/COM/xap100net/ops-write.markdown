
{%anchor write%}

# The Write Operation
{%section%}
{%column width=60% %}
In order to write objects to the Space, you use the write method of the GigaSpace interface. The write method is used to write objects if these are introduced for the first time, or update them if these already exist in the space. In order to override these default semantics, you can use the overloaded write methods which accept update modifiers such as WriteModifiers.UpdateOnly.
{%endcolumn%}
{%column width=35% %}
![POJO_write.jpg](/attachment_files/POJO_write.jpg)
{%endcolumn%}
{%endsection%}


#### PONO Example

The following example writes an `Employee` object into the space:

{% highlight csharp %}
Employee employee = new Employee ("Last Name", 32);
employee.FirstName="first name";
ILeaseContext<Employee> lc = spaceProxy.Write(employee);
Employee e = lc.Object;
{% endhighlight %}

#### SpaceDocument Example

Here is an example how you create a SpaceDocument, register it with the space and then write it into the space:

{%highlight csharp  %}
// Create the document
DocumentProperties properties = new DocumentProperties ();

properties.Add ("CatalogNumber", "av-9876");
properties.Add ("Category", "Aviation");
properties.Add ("Name", "Jet Propelled Pogo Stick");
properties.Add ("Price", 19.99f);
properties.Add ("Tags", new String[] { "New", "Cool", "Pogo", "Jet" });

DocumentProperties prop2 = new DocumentProperties ();
prop2.Add ("Manufacturer", "Acme");
prop2.Add ("RequiresAssembly", true);
prop2.Add ("NumberOfParts", 42);
properties.Add ("Features", prop2);

SpaceDocument document = new SpaceDocument ("Product", properties);

// Register the document
// Create type descriptor:
SpaceTypeDescriptorBuilder typeBuilder = new SpaceTypeDescriptorBuilder("Product");
typeBuilder.SetIdProperty("CatalogNumber");
typeBuilder.SetRoutingProperty("Category");
typeBuilder.AddPropertyIndex("Name");
typeBuilder.AddPropertyIndex("Price", SpaceIndexType.Extended);
ISpaceTypeDescriptor typeDescriptor = typeBuilder.Create();
// Register type descriptor:
spaceProxy.TypeManager.RegisterTypeDescriptor(typeDescriptor);

// Write the document into the space
ILeaseContext<SpaceDocument> lc = spaceProxy.Write (document);
{%endhighlight%}



#### Time To Live

To write an object into the space with a limited time to live you should specify a lease value (in millisecond). The object will expire automatically from the space.

{% highlight csharp %}
spaceProxy.Write(myObject, 10000)
{% endhighlight %}

{%anchor writeMultiple%}

#### Write Multiple
{%section%}
{%column width=60% %}
When writing a batch of objects into the space, these should be placed into an array to be used by the `ISpaceProxy.WriteMultiple` operation. The returned array will include the corresponding `ILeaseContext` object.
{%endcolumn%}
{%column width=35% %}
![POJO_write_multi.jpg](/attachment_files/POJO_write_multi.jpg)
{%endcolumn%}
{%endsection%}


#### Example

{% highlight csharp %}
Employee[] emps = new Employee[2];
emps [0] = new Employee ("Last Name A", 10);
emps [1] = new Employee ("Last Name B", 20);

ILeaseContext<Employee>[] leaseContexts = spaceProxy.WriteMultiple (emps);

for (int i = 0; i < leaseContexts.Length; i++) {
   Console.WriteLine ("Object UID " + leaseContexts [i].Uid + " inserted into the space");
}
{% endhighlight %}

{%note title=Here are few important considerations when using the batch operation:%}
-  should be performed with transactions - this allows the client to roll back the space to its initial state prior the operation was started, in case of a failure.
-  make sure `null` values are not part of the passed array.
-  you should verify that duplicated entries (with the same ID) do not appear as part of the passed array, since the identity of the object is determined based on its `ID` and not based on its reference. This is extremely important with an embedded space, since `WriteMultiple` injects the ID value into the object after the write operation (when autogenerate=false).

- Exception handling - the operation many throw the following Exceptions.
    - [WriteMultipleException](http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/Overload_GigaSpaces_Core_Exceptions_WriteMultipleException__ctor.htm)

{%endnote%}


#### Return Previous Value
{%section%}
{%column width=60% %}
When updating an object which already exists in the space, in some scenarios it is useful to get the previous value of the object (before the update). This previous value is returned in result `ILeaseContext.Object` when using the `WriteModifiers.ReturnPrevOnUpdate` modifier.
{%endcolumn%}
{%column width=35% %}
![write_return_prev-value.jpg](/attachment_files/write_return_prev-value.jpg)
{%endcolumn%}
{%endsection%}

{% highlight csharp %}
Employee employee = new Employee ("Last Name", 32);

ILeaseContext<Employee> lc = spaceProxy.Write(employee,WriteModifiers.ReturnPrevOnUpdate);
Employee previousValue = lc.Object;
{% endhighlight %}

{% info %}
Since in most scenarios the previous value is irrelevant, the default behavior is not to return it (i.e. `ILeaseContext.Object` return null). The `WriteModifiers.ReturnPrevOnUpdate` modifier is used to indicate the previous value should be returned.
{%endinfo%}

{%anchor asynchronousWrite%}

#### Asynchronous write
{%section%}
{%column width=60% %}
Asynchronous `write` operation can be implemented using a [Task](./task-execution-over-the-space.html), where the `Task` implementation include a write operation. With this approach the `Task` is sent to the space and executed in an asynchronous manner. The write operation itself will be completed once both the primary and the backup will acknowledge the operation. This activity will be performed as a background activity from the client perspective.
{%endcolumn%}
{%column width=35% %}
![write-oneway.png](/attachment_files/POJO_write_oneway.png)
{%endcolumn%}
{%endsection%}

#### Example

{% highlight csharp %}
Employee employee = new Employee ("Last Name", 32);
spaceProxy.Write(employee,WriteModifiers.OneWay);
{% endhighlight %}


#### Modifiers

For further details on each of the available modifiers see: [WriteModifiers](http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/P_GigaSpaces_Core_ISpaceProxy_WriteModifiers.htm)

{%note%}
Writing an object into a space might generate [notifications](./notify-container.html) to registered objects.
{%endnote%}

{% togglecloak id=os-write %}**Method summary...**{% endtogglecloak %}
{% gcloak os-write %}

Writes a new object to the space, returning its LeaseContext.{%netapi%}http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/Overload_GigaSpaces_Core_ISpaceProxy_Write.htm{%endnetapi%}

{%highlight csharp%}
ILeaseContext<T> Write(T entry);
ILeaseContext<T> Write(T entry, ITransaction tx, long lease, long timeout, WriteModifiers modifiers);
......

{%endhighlight%}

Writes new objects to the space, returning its LeaseContexts.{%netapi%}http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/Overload_GigaSpaces_Core_ISpaceProxy_WriteMultiple.htm{%endnetapi%}
{%highlight csharp%}
ILeaseContext<T>[] WriteMultiple(T[] entries);
ILeaseContext<T>[] WriteMultiple(T[] entries, ITransaction tx, long[] leases, WriteModifiers modifiers);
......
{%endhighlight%}

{: .table .table-bordered}
| Modifier and Type | Description | default |
|:-----|:------------|:-------- |
|T          | PONO, SpaceDocument||
|lease       |Time to live | Int64.MaxValue|milliseconds|
|timeout     | The timeout of an update operation, in milliseconds. If the entry is locked by another transaction wait for the specified number of milliseconds for it to be released. | 0  |
|[WriteModifiers](http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/P_GigaSpaces_Core_ISpaceProxy_WriteModifiers.htm)|Provides modifiers to customize the behavior of write operations | UpdateOrWrite  |
|[ILeaseContext](http://www.gigaspaces.com/docs/dotnetdocs{%currentversion%}/html/T_GigaSpaces_Core_ILeaseContext_1.htm) |LeaseContext is a return-value encapsulation of a write operation.| |
{% endgcloak  %}


