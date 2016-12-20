---
title: "What's New in ASP.NET Web API OData 5.3 | Microsoft Docs"
author: microsoft
description: ""
ms.author: riande
manager: wpickett
ms.date: 09/16/2014
ms.topic: article
ms.assetid: 
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/releases/whats-new-in-aspnet-web-api-odata-53
---
[Edit .md file](C:\Projects\msc\dev\Msc.Www\Web.ASP\App_Data\github\web-api\overview\releases\whats-new-in-aspnet-web-api-odata-53.md) | [Edit dev content](http://www.aspdev.net/umbraco#/content/content/edit/59683) | [View dev content](http://docs.aspdev.net/tutorials/web-api/overview/releases/whats-new-in-aspnet-web-api-odata-53.html) | [View prod content](http://www.asp.net/web-api/overview/releases/whats-new-in-aspnet-web-api-odata-53) | Picker: 59690

What's New in ASP.NET Web API OData 5.3
====================
by [Microsoft](https://github.com/microsoft)

This topic describes what's new for ASP.NET Web API OData 5.3.

- [Download](#download)
- [Documentation](#documentation)
- [OData Core Libraries](#corelib)
- [New Features](#newf)
- [Known Issues and Breaking Changes](#known-issues)
- [Bug Fixes](#bug-fixes)
- [ASP.NET Web API OData 5.3.1](#OD)

<a id="download"></a>
## Download

The runtime features are released as NuGet packages on the NuGet gallery. You can install or update to the released NuGet packages by using the NuGet Package Manager Console:

    Install-Package Microsoft.AspNet.OData -Version 5.3.0 
    Install-Package Microsoft.AspNet.WebApi.OData -Version 5.3.0

<a id="documentation"></a>
## Documentation

You can find tutorials and other documentation about ASP.NET Web API OData at the [ASP.NET web site](../odata-support-in-aspnet-web-api/index.md).

<a id="corelib"></a>
## OData Core Libraries

For OData v4, Web API now uses ODataLib version 6.5.0

<a id="newf"></a>
## New Features in ASP.NET Web API OData 5.3

### Support for $levels in $expand

You can use the $levels query option in $expand queries. For example:

    http://example.com/Employees?$expand=Manager($levels=2)

This query is equivalent to:

    http://example.com/Employees?$expand=Manager($expand=Manager))

<a id="open-entity-types"></a>
### Support for Open Entity Types

An *open type* is a stuctured type that contains dynamic properties, in addition to any properties that are declared in the type definition. Open types let you add flexibility to your data models. For more information, see xxxx.

### Support for dynamic collection properties in open types

Previously, a dynamic property had to be a single value. In 5.3, dynamic properties can have collection values. For example, in the following JSON payload, the `Emails` property is a dynamic property and is of collection of string type:

    {
       "Id": 1,
       "Name": "Ben",
       "Emails@odata.type": "#Collection(Edm.String)",
       "Emails": [
          "a@a.com",
          "b@b.com"
       ]
    }

### Support for inheritance for complex types

Now complex types can inherit from a base type. For example, an OData service could define the following complex types:

    public abstract class Shape
    {
        public bool HasBorder { get; set; }
    }
    
    public class Point
    {
        public int X { get; set; }
        public int Y { get; set; }
    }
    
    public class Circle : Shape
    {
        public Point Center { get; set; }
        public int Radius { get; set; }
    
        public override string ToString()
        {
            return "{" + Center.X + "," + Center.Y + "," + Radius + "}";
        }
    }
    
    public class Polygon : Shape
    {
        public IList<Point> Vertexes { get; set; }
        public Polygon()
        {
            Vertexes = new List<Point>();
        }
    }

Here is the EDM for this example:

[!code[Main](whats-new-in-aspnet-web-api-odata-53/samples/sample1.xml?highlight=8,15)]

For more information, see [OData Complex Type Inheritance Sample](http://aspnet.codeplex.com/SourceControl/latest#Samples/WebApi/OData/v4/ODataComplexTypeInheritanceSample/ReadMe.txt).

<a id="known-issues"></a>
## Known Issues and Breaking Changes

This section describes known issues and breaking changes in the ASP.NET Web API OData 5.3.

### OData v4

#### Query Options

Issue: Using nested $expand with $levels=max results in an incorrect expansion depth.

For example, given the following request:

    ~/Entities(6)?$expand=P($levels=2;$expand=D($levels=max))

If `MaxExpansionDepth` is 5, this query would result in an expansion depth of 6.

<a id="bug-fixes"></a>
## Bug Fixes and Minor Feature Updates

This release also includes several bug fixes and minor feature updates. You can find the complete list here:

- [Bug fixes](https://aspnetwebstack.codeplex.com/workitem/list/advanced?keyword=&status=All&type=All&priority=All&release=v5.3%20Beta&assignedTo=All&component=Web%20API|Web%20API%20OData&sortField=AssignedTo&sortDirection=Ascending&page=0&reasonClosed=Fixed)

<a id="OD"></a>
## ASP.NET Web API OData 5.3.1

In this release we made a [bug fix](https://aspnetwebstack.codeplex.com/workitem/list/advanced?keyword=&amp;status=All&amp;type=All&amp;priority=All&amp;release=v5.3.1%20Beta&amp;assignedTo=All&amp;component=Web%20API%20OData&amp;sortField=LastUpdatedDate&amp;sortDirection=Descending&amp;page=0&amp;reasonClosed=All) to some of the AllowedFunctions enums. This release doesn't have any other bug fixes or new features.