---
title: "Iteration #3 – Add form validation (VB) | Microsoft Docs"
author: microsoft
description: "In the third iteration, we add basic form validation. We prevent people from submitting a form without completing required form fields. We also validate emai..."
ms.author: riande
manager: wpickett
ms.date: 02/20/2009
ms.topic: article
ms.assetid: 
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/older-versions-1/contact-manager/iteration-3-add-form-validation-vb
---
[Edit .md file](C:\Projects\msc\dev\Msc.Www\Web.ASP\App_Data\github\mvc\overview\older-versions-1\contact-manager\iteration-3-add-form-validation-vb.md) | [Edit dev content](http://www.aspdev.net/umbraco#/content/content/edit/25237) | [View dev content](http://docs.aspdev.net/tutorials/mvc/overview/older-versions-1/contact-manager/iteration-3-add-form-validation-vb.html) | [View prod content](http://www.asp.net/mvc/overview/older-versions-1/contact-manager/iteration-3-add-form-validation-vb) | Picker: 27616

Iteration #3 – Add form validation (VB)
====================
by [Microsoft](https://github.com/microsoft)

[Download Code](iteration-3-add-form-validation-vb/_static/contactmanager_3_vb1.zip)

> In the third iteration, we add basic form validation. We prevent people from submitting a form without completing required form fields. We also validate email addresses and phone numbers.


## Building a Contact Management ASP.NET MVC Application (VB)
  

In this series of tutorials, we build an entire Contact Management application from start to finish. The Contact Manager application enables you to store contact information - names, phone numbers and email addresses - for a list of people.

We build the application over multiple iterations. With each iteration, we gradually improve the application. The goal of this multiple iteration approach is to enable you to understand the reason for each change.

- Iteration #1 - Create the application. In the first iteration, we create the Contact Manager in the simplest way possible. We add support for basic database operations: Create, Read, Update, and Delete (CRUD).

- Iteration #2 - Make the application look nice. In this iteration, we improve the appearance of the application by modifying the default ASP.NET MVC view master page and cascading style sheet.

- Iteration #3 - Add form validation. In the third iteration, we add basic form validation. We prevent people from submitting a form without completing required form fields. We also validate email addresses and phone numbers.

- Iteration #4 - Make the application loosely coupled. In this third iteration, we take advantage of several software design patterns to make it easier to maintain and modify the Contact Manager application. For example, we refactor our application to use the Repository pattern and the Dependency Injection pattern.

- Iteration #5 - Create unit tests. In the fifth iteration, we make our application easier to maintain and modify by adding unit tests. We mock our data model classes and build unit tests for our controllers and validation logic.

- Iteration #6 - Use test-driven development. In this sixth iteration, we add new functionality to our application by writing unit tests first and writing code against the unit tests. In this iteration, we add contact groups.

- Iteration #7 - Add Ajax functionality. In the seventh iteration, we improve the responsiveness and performance of our application by adding support for Ajax.


## This Iteration

In this second iteration of the Contact Manager application, we add basic form validation. We prevent people from submitting a contact without entering values for required form fields. We also validate phone numbers and email addresses (see Figure 1).


[![The New Project dialog box](iteration-3-add-form-validation-vb/_static/image1.jpg)](iteration-3-add-form-validation-vb/_static/image1.png)

**Figure 01**: A form with validation ([Click to view full-size image](iteration-3-add-form-validation-vb/_static/image2.png))


In this iteration, we add the validation logic directly to the controller actions. In general, this is not the recommended way to add validation to an ASP.NET MVC application. A better approach is to place an application s validation logic in a separate [service layer](http://martinfowler.com/eaaCatalog/serviceLayer.html). In the next iteration, we refactor the Contact Manager application to make the application more maintainable.

In this iteration, to keep things simple, we write all of the validation code by hand. Instead of writing the validation code ourselves, we could take advantage of a validation framework. For example, you can use the Microsoft Enterprise Library Validation Application Block (VAB) to implement the validation logic for your ASP.NET MVC application. To learn more about the Validation Application Block, see:

[<u>http://msdn.microsoft.<wbr>com/en-us/library/dd203099.<wbr>aspx</u>](https://msdn.microsoft.com/en-us/library/dd203099.aspx)

## Adding Validation to the Create View

Let s start by adding validation logic to the Create view. Fortunately, because we generated the Create view with Visual Studio, the Create view already contains all of the necessary user interface logic to display validation messages. The Create view is contained in Listing 1.

**Listing 1 - \Views\Contact\Create.aspx**

    <%@ Page Title="" Language="VB" MasterPageFile="~/Views/Shared/Site.Master" Inherits="System.Web.Mvc.ViewPage(Of ContactManager.Contact)" %>
    
    <asp:Content ID="Content1" ContentPlaceHolderID="head" runat="server">
    <title>Create</title>
    </asp:Content>
    
    <asp:Content ID="Content2" ContentPlaceHolderID="MainContent" runat="server">
    
        <%= Html.ValidationSummary() %>
    
        <% Using Html.BeginForm()%>
    
            <fieldset>
                <legend>Create New Contact</legend>
                <p>
                    <label for="FirstName">First Name:</label>
                    <%= Html.TextBox("FirstName") %>
                    <%= Html.ValidationMessage("FirstName", "*") %>
                </p>
                <p>
                    <label for="LastName">Last Name:</label>
                    <%= Html.TextBox("LastName") %>
                    <%= Html.ValidationMessage("LastName", "*") %>
                </p>
                <p>
                    <label for="Phone">Phone:</label>
                    <%= Html.TextBox("Phone") %>
                    <%= Html.ValidationMessage("Phone", "*") %>
                </p>
                <p>
                    <label for="Email">Email:</label>
                    <%= Html.TextBox("Email") %>
                    <%= Html.ValidationMessage("Email", "*") %>
                </p>
                <p class="submit">
                    <input type="submit" value="Create" />
                </p>
            </fieldset>
    
        <% End Using %>
    
    </asp:Content>

The field-validation-error class is used to style the output rendered by the Html.ValidationMessage() helper. The input-validation-error class is used to style the textbox (input) rendered by the Html.TextBox() helper. The validation-summary-errors class is used to style the unordered list rendered by the Html.ValidationSummary() helper.

> [!NOTE] 
> 
> You can modify the style sheet classes described in this section to customize the appearance of validation error messages.


## Adding Validation Logic to the Create Action

Right now, the Create view never displays validation error messages because we have not written the logic to generate any messages. In order to display validation error messages, you need to add the error messages to ModelState.

> [!NOTE] 
> 
> The UpdateModel() method adds error messages to ModelState automatically when there is an error assigning the value of a form field to a property. For example, if you attempt to assign the string "apple" to a BirthDate property that accepts DateTime values, then the UpdateModel() method adds an error to ModelState.


The modified Create() method in Listing 2 contains a new section that validates the properties of the Contact class before the new contact is inserted into the database.

**Listing 2 - Controllers\ContactController.<wbr />vb (Create with validation)**

    <AcceptVerbs(HttpVerbs.Post)> _
    Function Create(<Bind(Exclude:="Id")> ByVal contactToCreate As Contact) As ActionResult
        ' Validation logic
        If contactToCreate.FirstName.Trim().Length = 0 Then
            ModelState.AddModelError("FirstName", "First name is required.")
        End If
        If contactToCreate.LastName.Trim().Length = 0 Then
            ModelState.AddModelError("LastName", "Last name is required.")
        End If
        If (contactToCreate.Phone.Length > 0 AndAlso Not Regex.IsMatch(contactToCreate.Phone, "((\(\d{3}\) ?)|(\d{3}-))?\d{3}-\d{4}"))
            ModelState.AddModelError("Phone", "Invalid phone number.")
        End If        
        If (contactToCreate.Email.Length > 0 AndAlso  Not Regex.IsMatch(contactToCreate.Email, "^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$"))
            ModelState.AddModelError("Email", "Invalid email address.")
        End If
        If Not ModelState.IsValid Then
            Return View()
        End If
    
        ' Database logic
        Try
            _entities.AddToContactSet(contactToCreate)
            _entities.SaveChanges()
            Return RedirectToAction("Index")
        Catch
            Return View()
        End Try
    End Function

The validate section enforces four distinct validation rules:

- The FirstName property must have a length greater than zero (and it cannot consist solely of spaces)
- The LastName property must have a length greater than zero (and it cannot consist solely of spaces)
- If the Phone property has a value (has a length greater than 0) then the Phone property must match a regular expression.
- If the Email property has a value (has a length greater than 0) then the Email property must match a regular expression.

When there is a validation rule violation, an error message is added to ModelState with the help of the AddModelError() method. When you add a message to ModelState, you provide the name of a property and the text of a validation error message. This error message is displayed in the view by the Html.ValidationSummary() and Html.ValidationMessage() helper methods.

After the validation rules are executed, the IsValid property of ModelState is checked. The IsValid property returns false when any validation error messages have been added to ModelState. If validation fails, the Create form is redisplayed with the error messages.

> [!NOTE] 
> 
> I got the regular expressions for validating the phone number and email address from the regular expression repository at [<u>http://regexlib.com</u>](http://regexlib.com)


## Adding Validation Logic to the Edit Action

The Edit() action updates a Contact. The Edit() action needs to perform exactly the same validation as the Create() action. Instead of duplicating the same validation code, we should refactor the Contact controller so that both the Create() and Edit() actions call the same validation method.

The modified Contact controller class is contained in Listing 3. This class has a new ValidateContact() method that is called within both the Create() and Edit() actions.

**Listing 3 - Controllers\ContactController.<wbr />vb**

    Public Class ContactController
        Inherits System.Web.Mvc.Controller
    
        Private _entities As New ContactManagerDBEntities()
    
        Protected Sub ValidateContact(contactToValidate As Contact)
            If contactToValidate.FirstName.Trim().Length = 0 Then
                ModelState.AddModelError("FirstName", "First name is required.")
            End If
            If contactToValidate.LastName.Trim().Length = 0 Then
                ModelState.AddModelError("LastName", "Last name is required.")
            End If
            If (contactToValidate.Phone.Length > 0 AndAlso Not Regex.IsMatch(contactToValidate.Phone, "((\(\d{3}\) ?)|(\d{3}-))?\d{3}-\d{4}"))
                ModelState.AddModelError("Phone", "Invalid phone number.")
            End If        
            If (contactToValidate.Email.Length > 0 AndAlso  Not Regex.IsMatch(contactToValidate.Email, "^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$"))
                ModelState.AddModelError("Email", "Invalid email address.")
            End If
        End Sub
    
        '
        ' GET: /Contact
    
        Function Index() As ActionResult
            Return View(_entities.ContactSet.ToList())
        End Function
    
        '
        ' GET: /Contact/Create
    
        Function Create() As ActionResult
            Return View()
        End Function
    
        '
        ' POST: /Contact/Create
    
        <AcceptVerbs(HttpVerbs.Post)> _
        Function Create(<Bind(Exclude:="Id")> ByVal contactToCreate As Contact) As ActionResult
            ' Validation logic
            ValidateContact(contactToCreate)
            If Not ModelState.IsValid Then
                Return View()
            End If
    
            ' Database logic
            Try
                _entities.AddToContactSet(contactToCreate)
                _entities.SaveChanges()
                Return RedirectToAction("Index")
            Catch
                Return View()
            End Try
        End Function
    
        '
        ' GET: /Contact/Edit/5
    
        Function Edit(ByVal id As Integer) As ActionResult
            Dim contactToEdit = (from c in _entities.ContactSet _
                               where c.Id = id _
                               select c).FirstOrDefault()
    
            Return View(contactToEdit)
        End Function
    
        '
        ' POST: /Contact/Edit/5
    
        <AcceptVerbs(HttpVerbs.Post)> _
        Function Edit(ByVal contactToEdit As Contact) As ActionResult
            ' Validation logic
            ValidateContact(contactToEdit)
            If Not ModelState.IsValid Then
                Return View()
            End If
    
            ' Database logic
            Try
                Dim originalContact = (from c in _entities.ContactSet _
                                 where c.Id = contactToEdit.Id _
                                 select c).FirstOrDefault()
                _entities.ApplyPropertyChanges(originalContact.EntityKey.EntitySetName, contactToEdit)
                _entities.SaveChanges()
                Return RedirectToAction("Index")
            Catch
                Return View()
            End Try
        End Function
    
        '
        ' GET: /Contact/Delete/5
    
        Function Delete(ByVal id As Integer) As ActionResult
            Dim contactToDelete = (from c in _entities.ContactSet _
                               where c.Id = id _
                               select c).FirstOrDefault()
    
            Return View(contactToDelete)
        End Function
    
        '
        ' POST: /Contact/Delete/5
    
        <AcceptVerbs(HttpVerbs.Post)> _
        Function Delete(ByVal contactToDelete As Contact) As ActionResult
            Try
                Dim originalContact = (from c in _entities.ContactSet _
                                 where c.Id = contactToDelete.Id _
                                 select c).FirstOrDefault()
                _entities.DeleteObject(originalContact)
                _entities.SaveChanges()
                Return RedirectToAction("Index")
            Catch
                Return View()
            End Try
        End Function
    
    End Class

## Summary

In this iteration, we added basic form validation to our Contact Manager application. Our validation logic prevents users from submitting a new contact or editing an existing contact without supplying values for the FirstName and LastName properties. Furthermore, users must supply valid phone numbers and email addresses.

In this iteration, we added the validation logic to our Contact Manager application in the easiest way possible. However, mixing our validation logic into our controller logic will create problems for us in the long term. Our application will be more difficult to maintain and modify over time.

In the next iteration, we will refactor our validation logic and database access logic out of our controllers. We'll take advantage of several software design principles to enable us to create a more loosely coupled, and more maintainable, application.

>[!div class="step-by-step"] [Previous](iteration-2-make-the-application-look-nice-vb.md) [Next](iteration-4-make-the-application-loosely-coupled-vb.md)