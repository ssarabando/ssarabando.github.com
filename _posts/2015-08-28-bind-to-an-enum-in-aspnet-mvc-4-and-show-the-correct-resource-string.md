---
layout: post
title: Bind to an enum in ASP.NET MVC 4 and show the correct resource string
comments: False
---

# {{ page.title }}

_28 Aug 2015_

To my future self: this has been a very big detour. I'm again jumping over globalization hurdles, but this time in ASP.NET MVC 4.

My problem was simple: given an `enum` in one of the models, how to show it in a Razor view and in the correct language?

For example, given this enum:

``` cs
public enum SignalType
{
    Binary,
    Analog
}
```

How to make Razor render the following (simplified) HTML when the browser's UI is in portuguese:

``` html
<select>
    <option value="Binary">Binário</option>
    <option value="Analog">Analógico</option>
</select>
```

And this one when in english:

``` html
<select>
    <option value="Binary">Binary</option>
    <option value="Analog">Analog</option>
</select>
```

As usual, the plan was to use the `Display` attribute and fetch the string from a resource. Thus, the enum ended up like this:

``` cs
public enum SignalType
{
    [Display(ResourceType = typeof(Resources), Name = "SignalTypeBinaryCaption")]
    Binary,
    [Display(ResourceType = typeof(Resources), Name = "SignalTypeAnalogCaption")]
    Analog
}
```

Now I needed to iterate through the enum's members and render them as `option` tags.

To my disappointment, there was not built-in helper for that in MVC 4 (although it seems there is in ASP.NET MVC 5.1+) so I was forced to adapt a solution I found in Stack Overflow (see the references).

That solution creates a helper method named `EnumDropDownListFor<>` which we can call from a Razor view as usual but used a custom attribute named `Description` which wasn't what I wanted.
I needed the name from the resource file or from the `Display` attribute or the enum's value (if no `Display` attribute was present).

So I took the solution's code and changed the body of the `GetEnumDescription<TEnum>(TEnum value)` method like this:

``` cs
public static string GetEnumDescription<TEnum>(TEnum value)
{
    FieldInfo fi = value.GetType().GetField(value.ToString());

    var attributes = (DisplayAttribute[])fi.GetCustomAttributes(
        typeof(DisplayAttribute), false);

    if ((attributes != null) && (attributes.Length > 0))
    {
        return attributes[0].GetName();
    }

    return value.ToString();
}
```

If you compare the above with the original you'll notice that I only changed the attribute type being sought (`DisplayAttribute` instead of `DescriptionAttribute`) and invoked the 1st attribute's `GetName()` method instead of returning the `Description` property in the `if` statement.

With that I can now simply write the following in my Razor view:

``` cs
@Html.EnumDropDownListFor(model => model.SensorSignalType)
```

#### Full code listing

``` cs
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Linq.Expressions;
using System.Reflection;
using System.Web;
using System.Web.Mvc;
using System.Web.Mvc.Html;

namespace System.Web.Mvc.Html
{
    // See http://stackoverflow.com/a/5255108/215576
    public static class HtmlHelperExtensions
    {
        private static Type GetNonNullableModelType(ModelMetadata modelMetadata)
        {
            Type realModelType = modelMetadata.ModelType;

            Type underlyingType = Nullable.GetUnderlyingType(realModelType);
            if (underlyingType != null)
            {
                realModelType = underlyingType;
            }
            return realModelType;
        }

        private static readonly SelectListItem[] SingleEmptyItem = new[]
            {
                new SelectListItem { Text = "", Value = "" }
            };

        public static string GetEnumDescription<TEnum>(TEnum value)
        {
            FieldInfo fi = value.GetType().GetField(value.ToString());

            var attributes = (DisplayAttribute[])fi.GetCustomAttributes(
                typeof(DisplayAttribute), false);

            if ((attributes != null) && (attributes.Length > 0))
            {
                return attributes[0].GetName();
            }

            return value.ToString();
        }

        public static MvcHtmlString EnumDropDownListFor<TModel, TEnum>(
            this HtmlHelper<TModel> htmlHelper,
            Expression<Func<TModel, TEnum>> expression)
        {
            return EnumDropDownListFor(htmlHelper, expression, null);
        }

        public static MvcHtmlString EnumDropDownListFor<TModel, TEnum>(
            this HtmlHelper<TModel> htmlHelper,
            Expression<Func<TModel, TEnum>> expression,
            object htmlAttributes)
        {
            ModelMetadata metadata = ModelMetadata.FromLambdaExpression(
                expression, htmlHelper.ViewData);
            Type enumType = GetNonNullableModelType(metadata);
            IEnumerable<TEnum> values = Enum.GetValues(enumType).Cast<TEnum>();

            IEnumerable<SelectListItem> items = values.Select(value =>
                new SelectListItem
                    {
                        Text = GetEnumDescription(value),
                        Value = value.ToString(),
                        Selected = value.Equals(metadata.Model)
                    });

            // If the enum is nullable, add an 'empty' item to the collection
            if (metadata.IsNullableValueType)
                items = SingleEmptyItem.Concat(items);

            return htmlHelper.DropDownListFor(expression, items, htmlAttributes);
        }
    }
}
```

#### References

This solution was heavily influenced by the following post:

* [How do you create a dropdownlist from an enum in ASP.NET MVC?](http://stackoverflow.com/a/5255108/215576/)
