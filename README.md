# **Rails Forms**
## **Basic syntax**
```
<% form_with do |form| %>
    Form contents
<% end %>
```

## **Basic Search Form**

A basic search form has a label, a text input and a submit button.  

```
<%= form_with url: "/search", method: :get do |form| %>
    <%= form.label :query, "Search for:" %>
    <%= form.text_field :query %>
    <%= form.submit "Search" %>
<% end %>
```

### Result:
```
<form action="/search" method="get" accept-charset="UTF-8" >
  <label for="query">Search for:</label>
  <input id="query" name="query" type="text" />
  <input name="commit" type="submit" value="Search" data-disable-with="Search" />
</form>

```

## **Different Form Elements**
- `check_box` => type="checkbox"
- `radio_button` => type="radio"
- `text_area` => textarea tag
- `hidden_field` => type="hidden"
- `password_field` => type="password"
- `number_field` => type="number"
- `range_field` => type="range"
- `date_field` => type="date"
- `time_field` => type="time"
- `datetime_local_field` => type="datetime-local"
- `month_field` => type="month"
- `week_field` => type="week"
- `search_field` => type="search"
- `email_field` => type="email"
- `telephone_field` => type="tel"
- `url_field` => type="url"
- `color_field` => type="color"

## **Binding Form to model**
We can bind a model to the form using `model` argument.
```
<%= form_with model: @article do |form| %>
    <%= form.text_field :title %>
    <%= text_area :body, size: "60x10" %>
    <%= form.submit %>
<% end %>
```
#### Result:
```
<form action="/articles/42" method="post" accept-charset="UTF-8" >
  <input name="authenticity_token" type="hidden" value="..." />
  <input type="text" name="article[title]" id="article_title" value="My Title" />
  <textarea name="article[body]" id="article_body" cols="60" rows="10">
    My Body
  </textarea>
  <input type="submit" name="commit" value="Update Article" data-disable-with="Update Article">
</form>
```
1. Form fields automatically filled with corresponding values.
2. Field names are scoped with `article[...]`. So, we can access the values with `params[:article]` hash.

## **Record Identification in Rails**
 The Article model is directly available to users of the application, so - following the best practices for developing with Rails - you should declare it a resource:

```
resources :articles
```
When dealing with RESTful resources, calls to form_with can get significantly easier if you rely on record identification. In short, you can just pass the model instance and have Rails figure out model name and the rest:

#### **Creating a new article**
```
# long-style:
form_with(model: @article, url: articles_path)

# short-style:
form_with(model: @article)
```

#### **Editing an existing article**
```
# long-style:
form_with(model: @article, url: article_path(@article), method: "patch")

# short-style:
form_with(model: @article)
```
Record identification is smart enough to figure out if the record is new by asking `record.persisted?`.

## **PATCH, PUT and DELETE method working in Rails**
Most browsers don't support methods other than "GET" and "POST" when it comes to submitting forms. Rails works around this issue by emulating other methods over POST with a hidden input named "_method", which is set to reflect the desired method.

```
form_with(url: search_path, method: "patch")
```
#### Result:
```
<form accept-charset="UTF-8" action="/search" method="post">
  <input name="_method" type="hidden" value="patch" />
  <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  ...
</form>
```

## **Select Boxes**

Select Boxes in HTML requires lot of markup. Rails provides helper methods to reduce the burden.

```
<%= form.select :city, ["Berlin", "Chicago", "Madrid"] %>
<%= form.select :city, [["Berlin", "BE"], ["Chicago", "CHI"], ["Madrid", "MD"]], selected: "MD" %>
```

#### Result:
```
<select name="city" id="city">
  <option value="BE">Berlin</option>
  <option value="CHI">Chicago</option>
  <option value="MD">Madrid</option>
</select>

<select name="city" id="city">
  <option value="BE">Berlin</option>
  <option value="CHI">Chicago</option>
  <option value="MD" selected="selected">Madrid</option>
</select>
```

### **Option Groups**
For grouping the options, We can provide a hash.

```
<%= form.select :city,
      {
        "Europe" => [ ["Berlin", "BE"], ["Madrid", "MD"] ],
        "North America" => [ ["Chicago", "CHI"] ],
      },
      selected: "CHI" %>
```
#### Result:
```
<select name="city" id="city">
  <optgroup label="Europe">
    <option value="BE">Berlin</option>
    <option value="MD">Madrid</option>
  </optgroup>
  <optgroup label="North America">
    <option value="CHI" selected="selected">Chicago</option>
  </optgroup>
</select>
```
## ***`collection_select`*, *`collection_radio_buttons`* and *`collection_check_boxes`* Helpers**

- To generate a select box using a collection we can use `collection_select`.  
- To generate a set of radio buttons using a collection, we can use `collection_radio_buttons`.  
- To generate a set of check boxes using a collection, we can use `collection_check_boxes`.  


```
<%= form.collection_select :city_id, City.order(:name), :id, :name %>
<%= form.collection_radio_buttons :city_id, City.order(:name), :id, :name %>
<%= form.collection_check_boxes :city_id, City.order(:name), :id, :name %>
```

#### Result:
```
<select name="city_id" id="city_id">
  <option value="3">Berlin</option>
  <option value="1">Chicago</option>
  <option value="2">Madrid</option>
</select>

<input type="radio" name="city_id" value="3" id="city_id_3">
<label for="city_id_3">Berlin</label>
<input type="radio" name="city_id" value="1" id="city_id_1">
<label for="city_id_1">Chicago</label>
<input type="radio" name="city_id" value="2" id="city_id_2">
<label for="city_id_2">Madrid</label>

<input type="checkbox" name="city_id[]" value="3" id="city_id_3">
<label for="city_id_3">Berlin</label>
<input type="checkbox" name="city_id[]" value="1" id="city_id_1">
<label for="city_id_1">Chicago</label>
<input type="checkbox" name="city_id[]" value="2" id="city_id_2">
<label for="city_id_2">Madrid</label>
```

## **Uploading Files**
File upload fields can be rendered with the file_field helper. The most important thing to remember with file uploads is that the rendered form's enctype attribute must be set to "multipart/form-data". If you use form_with with :model, this is done automatically.

```
<%= form_with model: @person do |form| %>
  <%= form.file_field :picture %>
<% end %>

<%= form_with url: "/uploads", multipart: true do |form| %>
  <%= form.file_field :picture %>
<% end %>
```

Object in the `params` hash is an instance of `ActionDispatch::Http::UploadedFile`. 

> Note: Always declare the permitted parameters in the controller before passing to the model.
> ```
> def article_params
>   params.require(:article).permit(:title, :body)
> end
> ```


## **Resources**  
1. [Active View Form Helpers](https://guides.rubyonrails.org/v6.1/form_helpers.html#binding-a-form-to-an-object-the-fields-for-helper)