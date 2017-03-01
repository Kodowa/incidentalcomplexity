---
layout: post
title: "Example 3 - Jurassic Park"
author: "Corey Montella"
tags: []
---

![Example 3]({{ site.url }}/images/ex3.png)

### What is this?

This app demonstrates the use of a custom form component in a basic log in application. The app contains three views: a login form, a registration form, and a user profile form that displays the information entered during registration. These forms are built using a custom form component, which is defined at the end of the program. The form component allows for the concise definition of forms by defining common behavior like form submission and resetting.

### Application Set Up

The app contains the current page, as well as the current user. Initially, though, there is no user, so we just need to specify the current page.

```
bind @browser
  [#div #app class: "app-wrapper" page: "login" children: 
    [#div sort: 0 text: "Jurassic Park System Security Interface"]]
```

### Pages

#### Log In Form

The log in form contains two input boxes, one for the username and another for a password. We must explicitly sort the fields (using a `sort` attribute) to display them in a specific order.

```
search @browser
  app = [#app page: "login"]
  
bind @browser
  app.children +=
    [#div sort: 1 children:
      [#form name: "Login" sections: 
        [#section fields: 
          [#field sort: 2 type: "password" field: "password"]
          [#field sort: 1 type: "input" field: "username"]]]
      [#button #signup text: "Sign Up" ]]
```

A successful login is one where the username and password entered in the login form match some user/password combination stored in the system. For simplicity, passwords are stored as plain text, so we just need to search for a `#user` with a matching username and password. If one is found, we set it as the user attribute in the `#app` record.

```eve
search @browser @session
  [#form name: "Login" submission: [username password]]
  user = [#user username password]
  app = [#app]
    
commit @browser
  app.page := "profile"
  app.user := user
```

If the user enters a login that does not match a user, then display a message indicating that the login failed.

```eve
search @browser @session  
  form = [#form name: "Login" submission: [username password]]
  form-message = [#form-message form]
  
  not([#user username password])

commit @browser
  form-message.text := "** Login Failed **"
```

Clicking the sign up button changes the page to the sign up page

```
search @browser @event
  [#click element: [#signup]] 
  app = [#app]
  
commit @browser
  app.page := "signup"
```

#### Sign Up Form

The user registration page requests the name, department, a username and password.

```
search @browser
  app = [#app page: "signup"]
  
bind @browser
  app.children +=
   [#div children:
    [#form name: "Sign Up" options: [reset: true] sections: 
      [#section name: "User Info" fields: 
        [#field sort: 1 type: "input" field: "full-name"]
        [#field sort: 2 type: "input" field: "department"]]
      [#section name: "Account Info" fields: 
        [#field sort: 1 type: "input" field: "username"]
        [#field sort: 2 type: "password" field: "password"]
        [#field sort: 3 type: "password" field: "confirm-password"]]]
    [#button #login text: "Log In" ]]
```

We need to create a `#user` from the submission of the registration form. This will only work if every field has an entry, and the two password fields match.

```
search @browser
  [#form name: "Sign Up" submission: [username password confirm-password full-name department]]
  // The password and the confirmation must match
  password = confirm-password
  app = [#app]

commit @browser
  app.page := "login"
  
commit
  [#user username password name: full-name department]
```

Clicking the login button changes the page back to the login screen

```
search @browser @event
  [#click element: [#login]] 
  app = [#app]
  
commit @browser
  app.page := "login"
```

#### Profile Page

The profile page displays information relating to the current user profile. It is accessed after a successful submission of the login form, which creates a user attribute in the #app.

```
search @browser @session
  app = [#app page: "profile" user]
  
bind @browser
  app.children += 
  [#div class: "profile" children:
    [#div text: "Welcome {{user.username}}!"]
    [#div text: "Name: {{user.name}}"]
    [#div text: "Department: {{user.department}}"]
    [#button #logout text: "Log Out"]]
```

Clicking logout returns to the login page, and removes the user from `#app`.

```
search @browser @event
  [#click element: [#logout]] 
  app = [#app]
  
commit @browser
  app.user := none
  app.page := "login"
```


#### An easter egg

We can specify custom behavior by special casing search conditions and adding new side effects. In this block, we hijack the login process when the username is "dnedry". Instead of displaying the typical "login failed" message, we give the user a surprise.

```
search @browser @session
  app = [#app]
  
  [#form name: "Login" submission: [username password]]
  username = "dnedry"
  not([#user username password])
  
commit @browser
  app.children := none
  app.class -= "app-wrapper"
  app.class += "uh-uh-uh"
```

Clicking anywhere returns to the login screen

```
search @browser @session @event
  [#click]
  app = [#app class: "uh-uh-uh"]
  
commit @browser
  app.class += "app-wrapper"
  app.class -= "uh-uh-uh"
  app.children += [#div sort: 0 text: "Jurassic Park System Security Interface"]
  app.page := "login"
```

### A Custom Form Element

Forms have a title and one or more sections. Each section has an optional name, and contains one or more fields. Each field additionally has the input type of that field (input, radio button, drop down list, etc.).

A form starts as a `#form` record.

```eve
search @browser
  form = [#form]
  
bind @browser
  form += #div
  form.sort := 0
  form.class := "form"
  form.submission := []
```

Display the form name

```eve
search @browser
  form = [#form]
  
bind @browser
  form.children += [#div children:
    [#h1 class: "form-name" sort: 0 text: form.name]
    [#div #form-message form sort: 1 class: "form-message"]]

```

Display each section. To properly display sections, we need to add them to the children of the form.

```eve
search @browser
  form = [#form sections]
    
bind @browser
  form.children += [#div form section: sections class: "form-section" sort: 1]
  sections.form := form
```

If the section has a name, display it

```eve
search @browser
  section-display = [#div section]
  
bind @browser
  section-display.children += [#h2 class: "section-name" text: section.name, sort: 0]
```

Display the fields in each section. As we did with sections, to display fields we need to move them over to the children of the section display.

```eve
search @browser
  section-display = [#div section]
  field = section.fields
  
bind @browser
  section-display.children += 
    [#div field sort: field.sort form: section.form sort: 1 children: 
      [tag: field.type, placeholder: field.field, class: field.type]]
```

Display a submit button at the end of the form

```eve
search @browser
  form = [#form]
  
bind @browser
  form.children += [#button #submit form sort: 100 text: "Submit"]
```

Forms can have an optional reset button, which clears the fields in the form

```eve
search @browser
  form = [#form options: [reset: true]]
  
bind @browser
  form.children += [#button #reset form sort: 101 text: "Reset"]
```

Clicking the reset button clears each field in the form

```eve
search @event @browser
  [#click element: [#reset]]
  field-container = [#div field form]
    
commit @browser
  field-container.children.value := none
```

#### Save Input to Records

Form values are saved as a `#submission` when the submit button is clicked. This submission has a lifetime equal to that of the `#click`, so a submission must be committed to a record by the user. This allows the user to implement custom handling logic.

One thing this form component does not handle is form validation. In a future example, we will demonstrate how certain fields can be required, while others are optional. This form will submit any fields that are filled, while omitting any that are not. If a form is handled expecting fields that aren't submitted, then the submission will simply be ignored.

```eve
search @browser @event @session
  click = [#click element: [#submit form]]
  form = [#form submission]
  field-container = [#div field form]
  value = field-container.children.value
  key = field.field
  [#time timestamp: time]
  
bind @browser
  // When used in a bind or commit. lookup[] creates a record with the give attribute and value. We use it here to create a record with the attribute as the field name. 
  //For example, a login form with "username" and "password" fields could be accessed as [#form name: "Login" submission: [username password]]
  lookup[record: submission, attribute: key, value]
  field-container.children.value := none
```

#### Custom Input Types

Render password fields

```eve
search @browser
  password = [#password]
  
bind @browser
  password += #input
  password.type := "password"
  password.class := "password"
```

Render custom button styles

```eve
search @browser
  button = [#button]
  
bind @browser
  button.class += "button"
```

### Appendix

#### Test Data

```
commit
  [#user username: "jhammond" name: "John Hammond" department: "Executive" password: "password"]
  [#user username: "dnedry" name: "Dennis Nedry" department: "Engineering" password: "Mr. Goodbytes"]
  [#user username: "hwu" name: "Henry Wu" department: "Genetics" password: "slartibartfast"]
```

#### Styles


```css
{there is currently a bug that causes the first CSS block in an Eve program to be disregarded, so for a good time, leave this here}
```


```css
.application-container {
 background-color: #000; 
 color: green;
 font-family: monospace;
}

@media (min-width: 1800px) {
  .app-wrapper {
    background-image: url(http://i.imgur.com/BBPkd29.gif);
    background-repeat: no-repeat;
    width: 800;
    height: 658px;
    background-size: 100%;
    padding: 180px;
    padding-top: 130px;
  }
}

.profile {
 border: 1px solid green;
 padding: 10px;
 font-size:18px;
 border-radius: 5px;
}

.profile div {
 padding: 10px; 
}
  
.form-section {
 border: 1px solid green; 
 border-radius: 5px;
 padding: 10px;
 margin: 10px;
}

.form {
  border: 1px solid green; 
  border-radius: 5px;
  padding: 10px;
  margin: 10px;
  color: green;
}

.input {
 background-color: #000;
 border-radius: 5px;
 border: 1px solid green;
 padding: 5px;
 margin: 5px;
 font-family: monospace;
 color: green;
 outline: none;
}

.password {
 background-color: #000;
 border-radius: 5px;
 border: 1px solid green;
 padding: 5px;
 margin: 5px;
 font-family: monospace;
 outline: none;
 color: green;
}

.button {
  background-color: #000;
  color: green;
  border-radius: 5px;
  border: 1px solid green;
  padding: 5px;
  margin: 5px;
  cursor: pointer;
}

.form-message {
 color: green; 
}

.form-name {
  margin: 0px;
}

.section-name { 
  margin: 0px;
}

.uh-uh-uh {
 width: 320px;
 height: 520px;
 background-image: url(http://i.imgur.com/yz53s4N.gif); 
 background-color: #FFF;
}
```
