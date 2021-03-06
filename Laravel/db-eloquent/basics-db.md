# Database connexion

## Table of contents

* [Edit the .env](#Edit-the-.env)   
* [Create the db](#Create-the-db)  
* [Without eloquent](#Without-eloquent)  
* [With eloquent](#With-eloquent)  
* [Model binding](#Model-binding) 
* [Logic in model](#Logic-in-model)

## Edit the .env

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nameOfTheDatabase
DB_USERNAME=myUsername
DB_PASSWORD=myPassword
```

## Create the db

Enter in mysql : `mysql -u myUsername -p` afer that command we'll ask the password so type myPassword   
In mysql create de db : `create database nameOfTheDatabase;`   
If you want to add table, column and data, use phpMyadmin, adminer...
Notice : when creating a table remember to make it plurial. Example : if tou want your table to be named cat, you have to named it cats

## Without Eloquent

### In the controller

```php
public function show($id) {
        
    $seal = \DB::table('seal')->where('id', $id)->first();

    if (!$seal) {
        abort(404);
    }

    return view('seal', ['seal' => $seal]);
}
```

### In the view

```html
<p>A seal : {{ $seal->name }}</p>
```

## With Eloquent

### Create model

`php artisan make:model MyModel`   

The model created can be found at /app/Models and looks like : 
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Seal extends Model
{
    use HasFactory;
}
```

### In the controller

```php
public function show($id) {
    $seal = Seal::where('id', $id)->firstOrFail();
    return view('seal', ['seal' => $seal]);
}
```

Notice : `firstOrFail()` is a cleaner way to do an `->first()` and after a `abort(404)`

### Generating multiple files

With eloquent, you can generate multiple files when running the command for create a model.   
You can find the complete list with : `php artisan help make:model`

Some examples :    
I want to generate a controller and a migration with my model : `php artisan make:model MyModel -mc`   
I want to generate a controller with my model : `php artisan make:model MyModel -c`

### Function in model

In model you can add some function, for example : 
```php
public function complete() {
    $this->completed = true;
    $this->save();
}
```
This fonction modify the field complete of the model and pass it to true 


## Model Binding

### With Primary key

Model binding allow us to pass a model as argument of a controller's method, it'll get us the ressources depending the id pass in the route

The route : `Route::get('/article/{article}', [ArticlesController::class, 'show']);`

The controller : 
```php
public function show(Article $article) {
    // Before we use this but we do the exact same thing with the model binding
    // $article = Article::find($id);
    return view('article.article-details', ["article" => $article]);
}
```

### With any others fields

The previous point works only with primary key (id) if you want to use a slug or a title (for example) instead of an id, you can overwrite the method `getRouteKeyName()`

```php
 public function getRouteKeyName()
{
    return 'title';
}
```

Now you can type in your URL that : `http://127.0.0.1:8000/article/My first article` and it'll work !


## Logic in model

### Save

In some case, it's usefull to put logic in model instead of controller : when some action might be repeated or to clarify some actions

Examaple : in an Article model : 
```php
public function markAsRead() 
{
    // $this->read = true;
    // $this->save();

    // a condense way to do the same  : 
    $this->read->save(true)
}
```

### Sync

Sometime, when editing a value you not just edit a unique value, sometime you add a value to an array of other value (as with collection / pivot)   
When you want to replace the array entirely, I can use `sync`, it'll drop off all the current roles and replace it with `$role` :    
```php
// in User.php
public function assignRole($role) 
{
    $this->role->sync($role);
}
```

If you don't want to drop all of the user but just add the new one you can do this : 
```php
// in User.php
public function assignRole($role) 
{
    $this->role->syncWithoutDetaching($role, false);
    // same but less clear : $this->role->sync($role, false);
}
```

Notice : when seeing this we could thing that the same as the basic one with `save` but in fact not really. With `save`, it add the new `$role` but if you try to add a new `role` who already exist in the table it will crash and return an error. With `sync($role, false)`, it basically does the same thing but without throwing an error : it will add the `$role` in the table but if there already have the same `$role` in database, I'll do nothing.


### Logic using model instance 

In the previous point, the method `assignRole` accepted a model instance of Role, so you cannot just pass to it a simple sting as 'myRoleName'. You must instanciate the model : 

```php
// calling the method :
$user = User::find($id)
$manager = Role::firstOrCreate(['name' => 'manager']);*
$user->assignRole($manager);
```

But imagine doing this eatch time you call the `assignRole` method. Instead you can put some logic in the method :

```php
// in User.php
public function assignRole($role) 
{
    if(is_string($role)) {
        $role = Role::whereName($role)->firstOrFail();
    }
    $this->role->sync($role, false);
}
```

```php
// calling the method :
$user = User::find($id)
$user->assignRole('manager');
```