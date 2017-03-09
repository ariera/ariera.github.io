---
layout: post
title:  "Using Postgre's UUIDs in Laravel and Eloquent"
date:   2017-03-1 19:06:56 +0100
categories: laravel postgres
---

One of the things I love from postgres is the ability use **UUIDs as primary keys**. [Tom Harrison Jr. wrote recently a summary worth reading about the pros and cons of this approach](https://tomharrisonjr.com/uuid-or-guid-as-primary-keys-be-careful-7b2aa3dcb439#.poi02at77). For me the pros outweight the cons, but that's something you may have to decide by your own and on a project basis.

However, if you decide to go for it and you are using Laravel you'll be surprised to see that getting UUIDs to work **is not as easy as the official documentation makes you think.**

The following is a small cheatsheet for the things you have to take care of

## Setting up the db
The first thing you need is to activate the UUID extension in postgres. Simply create a new migration that looks like this:

{% highlight php %}
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
use Illuminate\Support\Facades\DB;

class AddUuidExtensionToPostgresql extends Migration
{

    public function up()
    {
        DB::statement('CREATE EXTENSION IF NOT EXISTS "uuid-ossp";');
    }

    public function down()
    {
        DB::statement('DROP EXTENSION IF EXISTS "uuid-ossp";');
    }
}
{% endhighlight %}

## Migrations
Let's say you want to have a `users` table. You probably want your `id`s to be automatically generated. Most of the tutorials I have found rely on `beforeCreate` callbacks on the model, at the applications level. I think it is best if the database takes care of this.

Unfortunately the nice `Schema` DSL doesn't provide a way to do this, so we need to use plain SQL for it.

{% highlight php %}
<?php
Schema::create('users', function (Blueprint $table) {
    $table->uuid('id');
    $table->primary('id');
    // ...
});
DB::statement('ALTER TABLE users ALTER COLUMN id SET DEFAULT uuid_generate_v4();');
{% endhighlight %}


## Models
Now let's create the corresponding `User` model.

{% highlight php %}
<?php

namespace App;
use Illuminate\Database\Eloquent\Model;
class User extends Model
{
  protected $casts = [
    'id' => 'string'
  ];
  protected $primaryKey = "id";
}
{% endhighlight %}

Couple of caveats here. First you need to **make sure that your primary key is typecasted as `string`**, otherwise it will be typecasted to `integer` giving you all sorts of funny errors.

Second: in this example I am naming my primary key `id`, which is Eloquent's default name. So overriding the `$primaryKey` in this particular example is not very interesting, but many people opt for naming their pks `uuid`, so pay attention to that if that's your case.

Lastly, many comments online and tutorials indicate that the public attribute `$incrementing` should be set to `false`, arguing that UUIDs don't need to be autoincremented. As much sense as that might make it managed to broke my applications. **Disabling the incrementing property somehow nullifies the `id` attribute in newly created records**, as the following code highlights.

{% highlight php %}
<?php
class User extends Model
{
  public $incrementing = false;
  // ...
}

$user = new User;
$user->save();
$user->id; //=> null
{% endhighlight %}

## Summary

Having UUIDs as primary keys for your records can be tricky to do right on Laravel for the first time. Hopefully this tips will help you avoid the beginners gotchas that I fell on :D
