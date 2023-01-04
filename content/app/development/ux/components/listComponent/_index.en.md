---
title: List component
linktitle: List component
toc: false
weight: 50
---

The list component is used to present data to the user in a table. Every row in the table is selectable. The component 
is supporting search, sorting and pagination. 

{{%notice warning%}}
This is new functionality. 
The setup must be done manually as of today. Support for setup through Altinn Studio will be launched shortly.
{{%/notice%}}

![ListComponent](listComponent.png "Eksempel på hvordan listekomponenten ser ut")

## How to define the component in code 
The component is of type `List`. One example on defintion of list component in layout.json:
```json
{
    "id": "list-component",
    "type": "List",
    "textResourceBindings": {
        "title": "Hvem gjelder saken?"
    },
    "dataModelBindings": {
        "name": "SelectedItem",
        "profession": "SelectedItemProfession"
    },
    "bindingToShowInSummary": "SelectedItem",
    "dataListId": "people",
    "tableHeaders": [ "Navn", "Alder", "Yrke" ],
    "sortableColumns": [ "Alder" ],
    "pagination": {
        "alternatives": [ 5, 10 ],
        "default": 5
    },
    "required": "true"
},
```

## Populating the table
The list component is populated with dynamic data. The data is defined in a seperate file which implements the interface
`IDataListProvider`. This works similar to code lists/options. The field `dataListId` defines which data list the component refers to.
The dynamic data lists kan be either open or secured. 
Here is one example of a class that implements the interface `IDataListProvider`

```csharp
public class ListCases : IDataListProvider
{
    public string Id { get; set; } = "people";

    public async Task<DataList> GetDataListAsync(string language, Dictionary<string, string> keyValuePairs)
    {
        int start = 0;
        int count = 10;

        if (keyValuePairs.ContainsKey("size") && keyValuePairs.ContainsKey("page"))
        {
            string size = keyValuePairs["size"];
            string page = keyValuePairs["page"];

            start = int.Parse(size) * int.Parse(page);
            count = int.Parse(size);
        }

        List<ListItem> items = new List<ListItem>();

        items.Add(new ListItem { Name = "Caroline", Age = 28, Profession = "Utvikler" });
        items.Add(new ListItem { Name = "Kåre", Age = 37, Profession = "Sykepleier" });
        items.Add(new ListItem { Name = "Johanne", Age = 27, Profession = "Utvikler" });
        items.Add(new ListItem { Name = "Kari", Age = 56, Profession = "Snekker" });
        items.Add(new ListItem { Name = "Petter", Age = 19, Profession = "Personlig trener" });
        items.Add(new ListItem { Name = "Hans", Age = 80, Profession = "Pensjonist" });
        items.Add(new ListItem { Name = "Siri", Age = 28, Profession = "UX designer" });
        items.Add(new ListItem { Name = "Tiril", Age = 40, Profession = "Arkitekt" });
        items.Add(new ListItem { Name = "Karl", Age = 49, Profession = "Skuespiller" });
        items.Add(new ListItem { Name = "Mette", Age = 33, Profession = "Artist" });

        if (keyValuePairs.ContainsKey("sortDirection"))
        {
            string sortDirection = keyValuePairs["sortDirection"];
            if (sortDirection == "asc")
            {
                items = items.OrderBy(o => o.Age).ToList();
            }
            else if (sortDirection == "desc") 
            {
                items = items.OrderBy(o => o.Age).ToList();
                items.Reverse();
            }
        }

        DataListMetadata appListsMetaData = new DataListMetadata() { TotaltItemsCount = items.Count };

        List<object> objectList = new List<object>();
        items.ForEach(o => objectList.Add(o));

        return new DataList { ListItems = objectList.GetRange(start, count), _metaData = appListsMetaData };
    }
}
```
In this example the list is created in code, but it is also possible to call an API which returns that data to be presented in the table. 
If this API supports sorting, pagination and search, you can pass this variables to the API to avoid fetching unnecessary data. 

## Columns
The table columns is defined in the component with the field `tableHeaders`. It is an array of strings. To support multiple 
languages this array can be populated with keys from the language files. For the content of the cells to match the header, you 
need to make sure that the table data is structured in the same order as the table headers. 

## Sorting
In `layout.json` you define which columns should be sortable with the field `sortableColumns`.
This is an array of strings, and the strings you use here also has to be defined as a header in the field `tableHeaders`.
When a column is sortable the column header will have an arrow to show the direction of the sorting.
The sorting logic you will have to implement your self. The method `GetDataListAsync` takes the parameter `keyValuePairs` 
which contains `sortDirection` and `sortColumn` for this purpose. This logic can be implemented directly in the `GetDataListAsync`
method, like in the previous example, or you can forward the sorting parameters to an external API you use.

## Pagination
To define pagination you use the field `pagination`, which takes an object as value. In this object you define `alternatives` with
an array of numbers which represents the choices the user will have for number of rows in the table. In addition you define `default`
which is the default of how many rows to be shown in the list. In the same way as with sorting, the pagination logic need to be implemented
for the correct data to show. The parameter `keyValuePairs` consists of the values size and page for this.  

## Search
For a user to be able to search the list, you need to add a text field of type search. This text field is connected to a field in the data model, 
and then this value in the data model can be used for search. I the list component you need to add the field `mapping`.
An example of this: 
```json
"mapping": {
    "soknad.navn": "search"
}
```
The value of the data model field `soknad.navn` will then be sent as a key value pair with the key `search`. 
OBS! The `mapping` field can also be used to send any other query parameters. 

## How to store data in the data model
The selected row is stored in the data model. You choose which column/cell values to store. Each column is stored in a seperate field in the data model. 
This is defined with the field `dataModelBindings`: 
```json
"dataModelBindings": {
    "name": "SelectedItem",
    "profession": "SelectedItemProfession"
},
```

SelectedItem and SelectedItemProfesstion are fields in the data model, and name and professtion are properties in the model used to describe a row in this case.
In the example shown here it is created a model to present a row like this: 
```csharp
public class ListItem
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Profession { get; set; }
}
```

## List component in the summary
If you want to show the selected row from the list component in the form summary, it will be shown simply with only one value. 
Which value is defined with the property `bindingToShowInSummary`, and will look like this: 

![ListComponentSummary](listComponentSummary.png "List component in summary")


## Secured data lists
In the same way as with code lists, you can secure the data list if they contain sensitive data. You then use the interface
 `IInstanceDataListProvider` for the class, and add the `secure` boolean to the component in layout.json.